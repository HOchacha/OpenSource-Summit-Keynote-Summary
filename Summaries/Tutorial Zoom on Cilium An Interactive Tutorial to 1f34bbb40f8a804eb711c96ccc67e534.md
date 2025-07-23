# Tutorial: Zoom on Cilium: An Interactive Tutorial to Learn How to Reshape the Traffic... Adam Sayah

다중 선택: OSS 2022 JN

아래는 Adam Saya 님의 eBPF/XDP 워크숍 발표 핵심을 한국어로 정리한 내용입니다.

---

## 1. eBPF·XDP 소개

- **eBPF**: Linux 커널에 프로그래밍 가능 코드(“BPF 프로그램”)를 삽입해, 네트워크·파일 시스템·트레이싱 등을 실시간 처리
- **XDP**: eBPF 기반의 Fast‐Path(최초 드라이버 단계) 패킷 처리 프레임워크
    - 네트워크 드라이버 바로 위에서 실행 ⇒ L2/L3 단계에서 패킷 필터링·로딩밸런싱·DDoS 차단 등 수행
    - 기존 iptables/TC(netem)보다 훨씬 낮은 지연·CPU 오버헤드

## 2. XDP의 주요 활용 사례

1. **보안 필터링**
    - 특정 IP 차단 (DDOS·불법 스캔 차단)
    - 프로토콜 제한 (UDP만 허용 등)
2. **초고속 로드밸런싱**
    - L2/L3 단계에서 웹서버·서비스 간 라운드로빈 분산
    - Meta, Cloudflare, Istio 등도 내부적으로 XDP 활용해 수십~수백 배 성능 개선
3. **모니터링·트레이싱**
    - 패킷 도착/전송 통계 집계 → Prometheus 지표로 노출
    - 파일 오픈·시스템 콜 관찰 등

## 3. 워크숍 데모 요약

1. **IPv6 패킷 완전 차단**
    - XDP 프로그램을 커널에 로드한 뒤, 모든 IPv6 헤더 패킷을 `XDP_DROP`
    - `ping6` 요청이 즉시 “Destination unreachable”
2. **특정 클라이언트 IP 차단**
    - 클라이언트 IP를 eBPF 코드에 하드코딩 → 해당 IP로부터의 모든 패킷 드롭
    - 다른 IP의 클라이언트는 정상 통신
3. **로빈헬드 로드밸런싱**
    - XDP 레벨에서 랜덤으로 두 백엔드(A/B) IP 중 하나를 목적지로 변경 후 전달
    - 클라이언트 → 라우터(XDP) → 백엔드(A/B) → 클라이언트 응답 흐름 완성

## 4. Bumblebee: eBPF/XDP 실험 도구

- **Bumblebee**: 작은 C 파일만으로
    1. eBPF/XDP 코드 자동 컴파일
    2. OCI 이미지 패키징 (`bee build …`)
    3. 레지스트리 푸시 (`bee push …`)
    4. XDP 프로그램 로드 (`bee run …`)
- Docker처럼 간단한 커맨드로 eBPF/XDP 워크플로우를 표준화해 줌

---

이처럼 XDP를 이용하면 **커널 드라이버 단계**에서 초저지연으로 패킷을 필터링·로드밸런싱·모니터링할 수 있어, 일반 iptables/TC 대비 **10~100배 성능** 이점을 기대할 수 있습니다. 관심 있는 실제 코드 예제는 Bumblebee 프로젝트와 eBPF 튜토리얼 레포지토리에서 모두 확인해 보세요!