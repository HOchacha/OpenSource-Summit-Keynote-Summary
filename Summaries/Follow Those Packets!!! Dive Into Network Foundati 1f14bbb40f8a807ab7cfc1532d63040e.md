# Follow Those Packets!!! Dive Into Network Foundations For Everyone! - Marino Wijay, Komodor

다중 선택: OSS 2024 NA

## 1. 강연자 및 배경

- **발표자**: Marino ID, Commodore 엔지니어 (Kubernetes 신뢰성·네트워킹 전문가)
- **세션 구성**: 35분 워크숍
    1. 네트워킹 기초 (패킷·라우팅·DNS·SDN·네임스페이스)
    2. eBPF 개요

## 2. 데모 환경 구성

- **클러스터**: Kind
- **CNI**: Cilium
- **IaC**: Pulumi로 Star Wars 앱 배포
    - Death Star ×2, TIE Fighter, X-Wing
    - 네임스페이스 `empire`에 레이블 `app=empire` 할당

## 3. 실습: 방화벽 정책 적용

1. TIE Fighter → Death Star API 호출 성공
2. `NetworkPolicy` 생성
    - **허용 대상**: 레이블 `app=empire` 보유한 Pod
    - **차단 대상**: X-Wing
3. X-Wing 요청 시 타임아웃 발생 → 차단 검증

## 4. 네트워킹 핵심 개념

### 4.1. 주소 체계

- **MAC 주소**: NIC 고유 식별자 (Layer 2)
- **IP 주소**: 논리적 네트워크 식별자 (Layer 3)

### 4.2. 프로토콜 계층 (OSI)

1. 물리·데이터링크 (Ethernet, MAC)
2. 네트워크 (IP)
3. 전송 (TCP/UDP)
    - TCP: 3-way handshake, 신뢰성 보장
    - UDP: 비연결, 실시간 통신에 적합
4. 애플리케이션 (HTTP, gRPC 등)

### 4.3. 스위칭 vs 라우팅

- **같은 서브넷**: ARP → 스위치 레벨 통신
- **다른 서브넷**: 라우터가 Layer 2⇄3 경계 역할

### 4.4. DNS

- **레코드 종류**: A, PTR, CNAME
- **역할**: 이름⇄IP 매핑 → 클라우드·Kubernetes 서비스 연결 필수

## 5. 소프트웨어 정의 네트워킹(SDN)

- **터널링**: VXLAN/Geneve로 L2 도메인 추상화
- **Overlay 네트워크**: 글로벌·클라우드 간 확장 가능
- **Kubernetes CNI**: 네임스페이스별 브리지·veth 자동 관리

## 6. 네임스페이스 기반 컨테이너 네트워킹

- **Pod ≒ Linux Network Namespace**
- **구성 요소**:
    - 가상 이더넷 페어(veth)
    - 브리지(bridge)
- **CNI 플러그인**: 생성·삭제·IP 관리 자동화

## 7. 트러블슈팅 도구

- **물리 연결 확인**: LLDP/CDP
- **패킷 캡처·분석**: Wireshark, kubeshark
- **분산 트레이싱**: Jaeger, Hubble

## 8. eBPF 개요

- **개념**: 커널 내 Sandbox 프로그램 실행 (Verifier 검증)
- **메커니즘**: Kprobes/Tracepoints → BPF 맵 → 사용자 공간 전달
- **활용 사례**: Cilium CNI, 커널 레벨 방화벽·정책·모니터링

## 9. 요약

1. **Kubernetes 네트워킹**: CNI로 네임스페이스·브리지·IP 자동 관리
2. **네트워크 기본**: 패킷 계층, 라우팅, DNS, SDN 이해
3. **eBPF**: 커널 레벨 관찰 및 정책 집행 가능