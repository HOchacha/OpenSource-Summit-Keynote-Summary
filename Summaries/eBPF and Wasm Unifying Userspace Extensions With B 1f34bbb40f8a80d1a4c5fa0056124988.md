# eBPF and Wasm: Unifying Userspace Extensions With Bpftime - Yusheng Zheng, eunomia-bpf

다중 선택: KubeCon 2025 EU

## 1. 개요

- **주제**: 사용자공간 확장(extension)을 안전·효율적으로 실행하기 위한 새로운 eBPF 런타임 “BPFtime”
- **핵심 아이디어**
    1. **확장 인터페이스 모델**을 도입해 “최소 권한 원칙”에 기반한 세밀한 권한 제어
    2. eBPF 검증(verification)과 WebAssembly 모델의 장점을 결합
    3. 기존 커널 eBPF 생태계와 호환되면서, 사용자공간 확장 코드를 안전·고성능으로 실행

---

## 2. 배경과 문제의식

1. **소프트웨어 확장의 필요성**
    - 웹 서버·DB·Kubernetes 등 다양한 시스템에서 “플러그인”·“모듈”·“WebAssembly”로 기능을 동적으로 추가
    - 사용자·운영자가 핵심 코드 변경 없이 요구사항을 커스터마이즈할 수 있어야 함
2. **안전–상호연동–성능의 상충**
    - **상호연동(Interconnect)**: 플러그인이 호스트 변수·함수에 얼마나 깊숙이 접근할지
    - **안전(Safety)**: 플러그인 오류나 악성 코드가 호스트 전체를 중단시키지 않도록 격리
    - **성능(Efficiency)**: 런타임·컨텍스트 전환·검증 오버헤드 최소화
3. **기존 기법의 한계**
    
    
    | 기법 | 장점 | 단점 |
    | --- | --- | --- |
    | LD_PRELOAD, 모듈 | 성능 우수, 통합 간편 | 격리 없음→전체 시스템 크래시 가능 |
    | WebAssembly (WASI + Component Model) | 기능별 세분화된 인터페이스, Capability 기반 안전성 | 런타임 체크·데이터 복사 → 지연 발생 |
    | Subprocess/RPC | 강력 격리, 권한 최소화 | IPC 컨텍스트 전환 오버헤드, 복잡도 상승 |
    | 커널 eBPF | 검증 기반 안전·고성능 | 인터페이스 확장 어려움, 디버깅 난이도 |
    | WASM + eBPF | (각자 장점 결합하려는 시도) | 실험 단계, 생태계·툴체인 성숙도 낮음 |

---

## 3. 확장 인터페이스 모델 (Extension Interface Model)

- **“Capability” 단위로 세밀히 분리**
    - **State Capabilities**: 접근 허용 맵(Map)·메모리 등
    - **Function Capabilities**: 호출 허용 함수(호스트 API)
    - **Entry Capabilities**: 플러그인이 진입 가능한 엔트리 포인트
- **BPFtime 특성**
    - 각 확장마다 “정의된 인터페이스”만 바인딩 → 최소 권한 원칙 구현
    - BTF( eBPF Type Format) 기반 검증으로 메모리·무한 루프 안전 보장
    - 인터페이스 레벨 어노테이션으로 “sleepable”, “destructor” 등 고급 안전 속성 명시 가능

---

## 4. BPFtime 런타임 아키텍처

```
[Extension Code]
   ↓  (compile & linker)
[BPFtime Loader] → BTF Verifier → eBPF Bytecode
   ↓  (load)
[BPFtime Runtime] ── maps / helper API ──┬─ Kernel eBPF ecosystem
                                         └─ Userspace plugins

```

- **고려 사항**
    - 기존 커널 eBPF 컴파일·로드 툴체인과 호환
    - 사용자공간 전용 “ebpf_vm” 삽입으로 isolation 확보
    - 핵심 eBPF 안전 검증은 기존 커널 Verifier에 위임 → 오버헤드 최소화

---

## 5. 주요 사용 사례 & 성능

| Use Case | 비교 대상 | 오버헤드 | 비고 |
| --- | --- | --- | --- |
| Tracepoint 관찰 | BCC / bpftrace | 10× 속도↑ | 사용자공간 전용 검증 덕분 |
| eBPF map (hash) | libbpf | 근접 성능 | BPFtime Maps 사용 |
| 사용자 커널 함수 후킹 | uprobe / kprobe | 성능 우수 | 컨텍스트 전환 없음 |

---

## 6. 결론 및 향후 과제

- **장점**
    - capability 기반 인터페이스로 “최소 권한”·“안전 격리” 달성
    - 커널 eBPF 검증 활용으로 런타임 오버헤드 최소화
    - 기존 eBPF 생태계 호환: libbpf, BCC, bpftrace 등 그대로 사용
- **남은 과제**
    1. **사용성**: 인터페이스 정의·플러그인 개발 툴체인 고도화
    2. **안정성**: 장기 운용 중 발견된 엣지 케이스 보완
    3. **생태계 연계**: WebAssembly Component Model과의 상호운용성 검토

---

**참고·발표자료**

- GitHub “eunomia-bpf/BPFtime”
- OSBI 2025 논문 “BPFtime: Safely and Efficiently Running Userspace Extensions”