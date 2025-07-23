# BPF LSM - Updates and What next? - KP Singh, Google

다중 선택: Linux Security Summit 2023 NA

**Stephan Greber – “BPF-LSM: Linux Security Modules via eBPF” 요약**

---

### 1. 배경: 커널 내 보안 로깅과 제어 확장

- Google 보안팀의 “LSM 레이어 전체에 필요한 감사(audit) 데이터를 얻자” 제안
- 기존 `audit` 서브시스템 수정, 모든 LSM(SELinux, AppArmor 등)에 똑같이 패치 → 너무 복잡
- “eBPF + LSM” 조합으로 커널 내 보안 후킹·로깅·정책 적용을 간결히 하자는 아이디어 → KRSI(krsi, Kernel Runtime Security Instrumentation) 착안

### 2. 발전 과정 & 커뮤니티 협력

- LSM·eBPF 커뮤니티 간 초창기 충돌 (“LSM은 무조건 zero‐overhead여야” vs. “eBPF는 비‐root sandbox용”)
- “Landlock” 프로젝트(비‐root 파일시스템 샌드박스) 팀과 “LSM=eBPF” 팀이 트루스 체결
- “Treaty of Impedance”:
    - 모든 LSM 훅에 대한 *성능 개선*(static calls 도입)
    - eBPF-LSM 자체는 “lsm=off” 기본 → eBPF 프로그램 설치 시 활성화

### 3. eBPF-LSM 작동 원리

1. **eBPF 프로그램 작성**
    
    ```c
    SEC("lsm/file_protect")
    int BPF_PROG(check_file_protect, struct file *file) {
        // audit·정책 로직
        return 0;  // 허용
    }
    
    ```
    
2. **커널 로딩·검증·JIT**
    - Verifier: 메모리 안전·루프 검사
    - JIT: 네이티브 코드로 변환
3. **LSM 훅에 직접 결합**
    - ftrace‐trampoline→static call 패치
    - `call rax`→`call <direct address>`로 성능 회복

### 4. “Treaty of Impedance” 성능 최적화

- **Indirect→Static calls**
    - 기존 LSM 훅마다 x86 thunk(리턴 오버헤드) 발생 → eBPF-LSM도 동일 페널티
    - `static_call` API로 부팅 시 직접호출 인스트럭션으로 패치 → 수 % 성능 향상(bench)
- **빈 콜(callback) 제거**
    - “default” 호출에도 간단히 `return 0` → hot path 오버헤드
    - `static_key`와 결합해 hook이 활성화된 경우에만 호출

### 5. eBPF-LSM 적용 사례

- **`inode_setxattr` 기본 동작 부작용**
    - “default=0” 로깅 제어나 SELinux·Smack 연동 누락 → corner‐case 정책 무시 문제
- **프로그램 서명(곧 도입 예정)**
    - `bpf_prog_load()` 시 ELF 객체 서명 검증(IMA) 어려움
    - “Loader program” 방식: load syscall trace + 로더 자체 서명 → bpf helper로 kernel 내 검증
- **`ptrace` 대체·확장**
    - File Descriptor 전달(`pidfd_open`/`pidfd_getfd`) + `bpf_task_storage`로 유연 정책

### 6. 향후 과제

1. **추가 LSM 훅 성능개선**: 빈 콜→static call 전환
2. **“정책 없음(E_NO_DECISION)” 에러 코드**: default후킹 부작용 방지
3. **프로그램 서명 & Gatekeeper**: PKCS#7, IMA 연동—신뢰할 수 있는 bpf-program 로드
4. **새로운 syscall 인터셉션**: `clone3()`, `mount_setattr()`, `io_uring` 등 복합 인자 지원
5. **완전한 eBPF 기반 사용자공간 보안 정책 엔진**: LSM 프레임워크 의존 ↓

---

**결론**:

eBPF-LSM(“KRSI”)는 “커널‐내 보안 후킹·로깅·정책 적용”을 훨씬 유연하게 구현하며, “Treaty of Impedance”의 static call 최적화로 기존 LSM 성능 저하 문제를 해소합니다. 향후 인증·추가 시스템 콜 지원·정책 엔진 통합까지 발전할 예정입니다.