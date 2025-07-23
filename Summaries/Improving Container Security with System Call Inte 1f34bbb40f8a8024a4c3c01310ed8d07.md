# Improving Container Security with System Call Interception

다중 선택: Linux Security Summit 2023 NA

**발표 전체 요약**

1. **배경 및 목표**
    - 비특권(non-root) 컨테이너에서도 필요한 “특권 작업”을 안전하게 허용하고자 함
    - 단순히 `capabilities`·Namespace·LSM으로는 해결하기 어려운 사례들(디바이스 노드 생성, 파일시스템 마운트, `sysinfo` 정보 조회 등)을 다룸
2. **seccomp 시스템 콜 가로채기(user-notification) 개요**
    - `SECCOMP_FILTER_FLAG_NEW_LISTENER` 플래그로 필터를 설치 → 커널이 “해당 시스템 콜이 발생했다”는 알림 FD 발급
    - 사용자 공간 데몬이 FD를 `poll()` → `ioctl()`으로 시스템 콜 인자 확인
    - `CONTINUE`·`REJECT(EACCES)`·`ALLOW(EPERM)` 등으로 콜 결과 결정
    - 사용자 공간에서 직접 “대체 구현(emulation)”도 가능
3. **주요 인터셉트 대상 & 해결 과제**
    - **`mknod`, `setxattr`**:
        - 화이트아웃(overlayfs)·확장속성(xattrs) 생성 허용
    - **`mount`**:
        - block-fs 직접 마운트 → unprivileged 컨테이너 불가
        - `mount` 인터셉트 후 FUSE 기반 동적 위임(호스트 모듈 → 사용자 공간)으로 컨테이너 내 마운트 지원
    - **`sysinfo`**:
        - 호스트 전체 리소스 정보 노출 → `cgroups` 제한값 반영하여 격리된 컨테이너 정보만 리턴
4. **구현상의 유의점**
    - 포인터 인자(예: `open`, `mount` struct) 검사 → `process_vm_readv()` 등으로 안전하게 읽기
    - `CONTINUE` 모드 사용 시 “커널 방어”(ultimate security boundary)가 여전히 유효한지 확인
    - 컨테이너 관리자 데몬에 필요한 최소 권한만 부여하여 실행
5. **시연 데모**
    - `mknod` 인터셉트로 `/dev/null`, whiteout 생성
    - `mount`→FUSE 위임 예시 (`shiftfs` 방식으로 UID/GID 매핑)
    - `sysinfo` 가로채기로 `free`·`uptime` 명령에 컨테이너 한정 리소스 반영
6. **확장 및 향후 과제**
    - EBPF 기반 인터셉션으로 포인터·구조체 필터링 강화
    - `clone3`, `mount_setattr` 등 신규·멀티플렉서형 시스템 콜 지원
    - “컨테이너가 요구할 때만” 호스트 모듈 자동 로드(fan-out) 구현 검토

---

위 방식을 통해 “비특권 컨테이너” 환경에서도 안전하게 일부 특권 시스템 콜을 위임·제어할 수 있음을 보여주었습니다.