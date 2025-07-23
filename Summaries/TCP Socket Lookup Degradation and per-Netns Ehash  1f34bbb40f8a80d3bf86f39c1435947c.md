# TCP Socket Lookup Degradation and per-Netns Ehash - Kuniyuki Iwashima, Amazon Web Services

다중 선택: OSS 2022 JN

아래는 “TCP 스키마 조회(lookup) 병목 개선 및 네임스페이스별 E‐Hash” 발표 내용을 한국어로 정리한 것입니다.

---

## 1. 소켓 생성 시점 내부 구조

1. **getaddrinfo → socket()**
    - `socket()` 호출 시, 커널은 프로토콜(IPv4/TCP 등)별 `struct sock` 구조체를 할당하고 파일 디스크립터를 돌려줌
2. **bind() → B-Hash 등록**
    - 소켓에 IP:포트 바인딩 시, `bind()`가 충돌 여부를 `B-Hash`(키: 네임스페이스+로컬 포트)에서 검사
    - 문제 없으면 해당 버킷에 새 소켓을 연결(링크드 리스트)
3. **listen() → L-Hash 등록**
    - `listen()` 시 `struct sock` 상태를 `TCP_LISTEN`으로 바꾸고, `L-Hash`(키: 네임스페이스+로컬 포트)에도 연결
4. **connect() → E-Hash 등록**
    - 클라이언트 `connect()` 시, 미리 바인딩된 소켓이 없으면 커널이 포트를 선택해 `B-Hash` 검사 → 3-way handshake → 새 소켓을 `E-Hash`(키: 5-tuple)로 등록

> 정리:
> 
> - B-Hash: bind 충돌 확인
> - L-Hash: listen 충돌 확인
> - E-Hash: Established 연결 조회

---

## 2. B-Hash/L-Hash 조회 병목 문제

- **해시 키 단순화 ⇢ 큰 링크드 리스트**
    - (기존) B-Hash/L-Hash 키가 ‘네임스페이스 + 포트’(2-tuple)만 사용 → 같은 포트 바인딩 소켓이 모두 한 버킷에 몰림
    - 특히 `SO_REUSEPORT`, `SO_REUSEADDR` 옵션 사용 시 더욱 긴 리스트 ⇒ 조회(lookup) 성능 급감
- **L-Hash2 도입 (Linux 4.16)**
    - 키를 ‘네임스페이스 + 포트 + IP’(3-tuple)로 변경 → 서로 다른 IP 바인딩은 다른 버킷으로 분산
- **B-Hash2 도입 (Linux 6.1)**
    - 동일하게 ‘네임스페이스 + 포트 + IP’(3-tuple) 키를 사용해 B-Hash2 추가
    - *기존 B-Hash는 여전히 포트만 필터링할 때 필요(예: FTP 데이터 채널)*
- **효과:**
    - 조회 리스트 길이 획기적 단축 → `listen()`·`bind()` 지연 및 SYN 패킷 처리 지연 대폭 감소

---

## 3. 네임스페이스별 E-Hash (TCP netns E-Hash)

- **기존 E-Hash 단일 테이블**
    - 모든 네임스페이스의 5-tuple 소켓을 하나의 전역 해시 테이블(E-Hash)에 저장 ⇒ namespace 충돌·조회 병목 발생
- **TCP netns E-Hash 도입 (Linux 6.1)**
    - 각 네트워크 네임스페이스별로 별도 E-Hash 테이블 제공 ⇒ 전역 테이블 조회 병목 해소
    - 조회 키는 ‘네임스페이스 + 5-tuple’
- **조정 가능 매개변수**
    1. `net.ipv4.tcp_child_netns_ehash_entries`
        - 새 네임스페이스 생성 시 E-Hash 크기(항목 수) 지정(2의 거듭 제곱으로 자동 반올림)
        - 기본 0 ⇒ 전역 E-Hash만 사용
    2. `net.ipv4.tcp_child_netns_ehash_entries_current`
        - 현재 네임스페이스 E-Hash 크기 확인(음수면 전역 E-Hash 사용)
- **NUMA 고려**
    - 전역 E-Hash: 부팅 시 모든 NUMA 노드에 분산
    - netns-E-Hash: 네임스페이스 생성 시 로컬 NUMA 노드에만 할당 ⇒ 대규모 멀티테넌트 환경에서 추가 조정 필요

---

### 요약

- **B-Hash2, L-Hash2**: 키에 IP 추가로 포트별 소켓 분산 → `bind()`/`listen()` 병목 완화
- **TCP netns E-Hash**: 네임스페이스별 독립 해시 테이블로 SYN/E-Hash 조회 병목 제거
- **튜닝 포인트**: 해시 크기(`tcp_child_netns_ehash_entries`), NUMA 정책

이를 통해 대규모·멀티테넌트 환경에서 TCP 연결 설정 및 조회 비용을 효과적으로 줄일 수 있습니다.