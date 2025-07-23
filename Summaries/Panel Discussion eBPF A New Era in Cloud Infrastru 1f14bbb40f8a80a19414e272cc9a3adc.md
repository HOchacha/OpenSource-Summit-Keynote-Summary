# Panel Discussion: eBPF: A New Era in Cloud Infrastructure Tools - Liz Rice, Isovalent; Frederic...

다중 선택: OSS 2024 EU

## 1. 패널 구성

- **사회**: Richard Simon (T-Systems CTO)
- **패널**:
    - Liz Rice (Cisco, CTO 오픈소스 담당)
    - Frederick Branik (Polar Signals 창립자)
    - Hean Malau (DataDog 시니어 SW 엔지니어)
    - Yen Jang (Eunomia Inc. 오픈소스 개발자)

## 2. eBPF란?

- 커널 내 “프로그램 삽입” 서브시스템
- 다양한 이벤트(함수 호출, 패킷 입출력, 시스템콜 등)에 후킹 가능
- **주요 활용처**: 관측성(Tracing/Profiling), 네트워킹, 보안 정책, 스케줄러 제어

## 3. 핵심 강점 & 성능

- **User-kernel 컨텍스트 스위치 최소화** ⇒ 극소 오버헤드
- 커널 내 필터링·집계로 “필요 데이터만” user-space 전송
- Netflix·Meta 등 대규모 프로덕션에서 수년간 활용
- 벤치마크 예시: Linux 커널 빌드 중 600만 파일 읽기 관측 시 CPU 오버헤드 ≲2%

## 4. 운영 시 도전 과제

- **BPF Verifier 호환성**: 커널 버전별 제약→CI에 다양한 버전 테스트 필요
- **특권 필요**: 프로그램 로딩 권한 확보(보안 검토)
- **복잡도**: 직접 커널 구조체를 다루는 진입 장벽

## 5. 개발 진입 장벽 완화

- **고수준 도구**:
    - `bpftrace` oneliner DSL (즉시 사용 가능한 프로파일·트레이스)
    - Cilium, Tetragon, Falco, BCC, libbpf 등 생태계
- **커뮤니티·자료**: eBPF Summit / eBPF Day / Linux Plumbers 영상·슬라이드
- **온라인 지원**: ebpf.io, eBPF Slack, GitHub 오픈소스 리포지터리

## 6. 차세대 eBPF 전망

- **스케줄러 확장**: Google SkyDive/XDP 기반 커널 스케줄러 튜닝
- **Windows 지원**: Microsoft ‘eBPF for Windows’ 프로젝트
- **User-space tracing 최적화**: 더 낮은 오버헤드·정밀도 향상
- **경쟁·보완 기술**: WebAssembly(격리 런타임) vs. eBPF(커널 확장)

---

*“eBPF는 단순히 패킷 필터가 아니라, 운영체제 그 자체를 동적으로 재구성하는 혁신 플랫폼!”*