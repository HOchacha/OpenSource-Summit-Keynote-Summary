# Mix Kubernetes with Linux Kernel Tracing to Boost the Observability of Your... - Tzvetomir Stoyanov

다중 선택: OSS 2023 EU

## 1. 요구사항

- **유저스페이스 레벨에서** 동작
- **컨테이너 단위**로, Kubernetes 같은 환경에서도 사용
- **커널 모듈 설치 없이**(BPF 코어 기능은 쓰되, 직접 커널 모듈 빌드·배포 없이)

## 2. 대표적인 솔루션들

1. **Jaeger / Pixie / Hubble** 등 클라우드 네이티브 분산트레이싱
    - HTTP/gRPC 호출, 마이크로서비스 간 상호작용 중심
    - 애플리케이션 레벨(라이브러리 인스트루멘테이션)
2. **BCC 기반 도구들** (`opensnoop`, `execsnoop` 등)
    - 유저스페이스 파이썬 스크립트 + 커널 BPF 코드
    - 즉시 컴파일되고, per-CPU buffer 쓰는 전통적 방식
3. **BPF Curry + Bumblebee**
    - 사전 컴파일(`compile once`) → 여러 배포 환경에서 동일 바이너리 사용
    - Ring Buffer + OCI 이미지 형태로 배포하여 커널 모듈 설치 불필요
    - Kubernetes DaemonSet으로 운영하고, Prometheus 메트릭으로 노출 가능

## 3. 주요 데모 흐름 (Bumblebee 활용 예)

1. **`opensnoop`** 유사 코드 변환
    - Perf Buffer → Ring Buffer로 변경
    - 라이브러리 의존 없이 `bumblebee build` → OCI 이미지 생성
2. **로컬 레지스트리에 업로드**
3. **DaemonSet으로 k8s에 배포**
4. **Pod(Single container) 생성 후 파일 열기/닫기 트레이스**
5. **Prometheus+Grafana** 연동하여 메트릭 시각화

## 4. 성능 이슈 및 고려사항

- BPF 자체 오버헤드: 보통 CPU 사용량 1–2% 수준
- Ring Buffer 공유 버퍼 덕분에 per-CPU Perf Buffer 대비 메모리/성능 개선
- OCI 이미지 방식으로 배포 → 커널 모듈 빌드·배포 없이 운영

## 5. 결론

- **유저스페이스 전용 BPF 추적**은 Bumblebee 같은 도구로 충분히 가능
- **컨테이너 환경**에 잘 맞도록 DaemonSet + Prometheus 연동까지 쉽게 구성
- **성능·운영 부담**도 비교적 적고, 커널 모듈 관리 없이 배포·업데이트 용이