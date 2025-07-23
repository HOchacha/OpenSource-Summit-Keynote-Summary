# Secure the Linux Kernel with eBPF Linux Security Module - Vandana Salve, Independent Consultant

다중 선택: OSS 2023 EU

**“BPF 기반 LSM으로 Linux 커널 보호하기” 요약 (Wandera, Micron)**

---

### 1. 배경: Linux 보안 모델

- **기본 보안**: 물리적 잠금 → 사용자·파일 권한 → 감사(audit) → 취약점 패치
- **LSM( Linux Security Modules)**:
    - **목적**: MAC(Mandatory Access Control) 구현
    - **작동원리**: 커널 주요 제어경로(syscalls 등)에 “보안 훅” 삽입 → 정책 검증·에러 반환
    - **한계**: 커널 내장→정책 변경·업그레이드 시 재빌드·재부팅 필요

### 2. eBPF+LSM 통합

- **BPF LSM(5.7+)**:
    - LSM 훅을 **eBPF 프로그램**으로 대체→커널 외부에서 정책 배포·변경 가능
    - **장점**
        1. **런타임 로드/언로드**: 재부팅·커널 패치 없이 즉시 정책 적용
        2. **안전성**: eBPF verifier가 무한루프·메모리 안전성 검사
        3. **유연성**: 사용자 공간 코드(Go/C)로 보안 로직 구현

### 3. 구현 예시: `setpriority` 제어

- **목표**: 프로세스가 **비정상적 우선순위(음수 nice 값)** 설정 못하도록 차단
- **방법**
    1. **eBPF 프로그램** (C 코드):
        
        ```c
        SEC("lsm/task_setnice")
        int BPF_PROG(check_nice, struct task_struct *p, long nice) {
            return (nice < 0) ? -EPERM : 0;
        }
        
        ```
        
    2. **로더(Go)**: eBPF 바이너리 로드→`BPF_PROG`를 `task_setnice` LSM 훅에 연결
    3. **결과**: nice < 0 요청 시, 즉시 `EPERM` 반환 → 우선순위 변경 차단

### 4. 전체 흐름

1. **컴파일**: eBPF 프로그램→바이트코드
2. **로딩**: 사용자 공간 로더가 `bpf()` syscall로 커널에 삽입
3. **검증**: eBPF verifier가 안전성 검증
4. **실행**: `task_setnice` syscall 진입 시 eBPF 훅 호출→정책 적용

### 5. 결론

- **BPF LSM** 융합으로
    - 커널 재시작 없이 **정책 배포·변경** 가능
    - eBPF의 **검증·격리** 이점 활용
    - **동적·세분화된** 보안 통제 실현
- **활용 포인트**:
    1. 시스템 콜·네트워크·파일 I/O 등 **다양한 훅**에 적용
    2. eBPF로 작성된 LSM 훅에 “정책 로직” 집중
    3. eBPF 프로그램 로더를 통해 **운영 환경 반영**

> 추가 자료:
> 
> - Kernel v5.7+ “BPF LSM” 기능
> - `libbpf` 예제: LSM 훅 연결
> - eBPF verifier 동작원리

질문 환영합니다!