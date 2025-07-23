# Using eBPF for Non-invasive, Performant, Instant Network Monitoring - Mario Macías & Marc Tudurí

다중 선택: KubeCon 2025 EU

## eBPF 기반 비침습적(Non-Invasive) 실시간 네트워크·애플리케이션 모니터링

### 1. eBPF 활용 개요

- **eBPF**: 커널 공간에 안전한 사용자 프로그램 삽입
    - **JIT 컴파일** + **Verifier** → 성능·안전성 보장
    - **Maps** 통해 커널·유저공간 간 데이터 교환
- **다양한 eBPF 프로그램**
    - **kprobe/uprobe**: 커널·사용자 함수 호출 시점 후킹
    - **TC/BPF**: 네트워크 패킷 입출력 레벨 후킹
    - **XDP**: NIC 드라이버 바로 위에서 초고속 패킷 처리

### 2. Grafana Baila 솔루션

- **Zero-code 자동 계측**: 애플리케이션 재배포·코드 수정 불필요
- **수집 데이터**
    - **네트워크 메트릭** (L3/L4) → `netflow_bytes` 등
    - **애플리케이션 메트릭** (OpenTelemetry 규격)
    - **애플리케이션 트레이스** (HTTP/gRPC/Kafka 등)
- **문제 해결**:
    1. **분산 이벤트 합산**
        - TCP 소켓·TLS 프레임·사용자 스레드 이벤트 → 맵에 저장 → User-space에서 병합
    2. **컨텍스트 전파**
        - HTTP 헤더 불가 → IPv4 Option에 Trace-ID 삽입
        - 수신 시 TCP Len/ACK 활용해 완전한 Trace 재구성

### 3. Kubernetes 연동

- **Informer** 이용해 Pods/Services/Nodes 이벤트 구독
- **프로세스 → Pod 맵핑**
    - `/proc/[pid]/cgroup` 분석 → 컨테이너 ID → 해당 Pod 식별
- **Zone & 외부 트래픽**
    - Node Label(`topology.kubernetes.io/zone`) → Zone 간 흐름 집계
    - DNS 패킷 후킹 → `netflow`에 호스트명(enriched DNS) 추가
- **메트릭 카디널리티 제어**
    - Prometheus 레이블 `include`/`exclude` 설정으로 폭발 억제

### 4. 앞으로의 방향

- **OpenTelemetry eBPF 계측 커뮤니티 기여**(Grafana Baila 기부 진행 중)
- **언어·프레임워크 확장**: Go/Rust/Python/Java 등 자동 계측 지원 강화
- **IPv6, Parquet·메타데이터** 등 추가 프로토콜·저장 옵션 연구

---

**문의·피드백**:

- GitHub → `grafana/baila`
- Slack → #ebpf-otel-auto-instrumentation
- 🗓️ 매주 미팅 (donation 이후 공지 예정)