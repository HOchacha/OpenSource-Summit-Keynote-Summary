# Containers Security Layers and How Not to Break them - Aviv Sasson, Palo Alto Networks

다중 선택: OSS 2022 NA

**발표 요약: “컨테이너 보안 레이어”**

1. **컨테이너 기본**
    - OS 가상화: 이미지로 앱·의존성 묶어 배포
    - 호스트 커널 공유→VM 대비 경량·고성능
    - 네임스페이스(namespaces) × cgroups로 프로세스 격리·자원 제한
    - 보통 비(非)root 권한으로 실행
2. **제한 사항 & “추가 권한” 필요 사례**
    - 기본 정책으로는 아래 작업 불가
        - 커널 모듈(load/unload)
        - eBPF 프로그램 로드
        - 고급 네트워킹(iptables, raw sockets)
        - Docker‐in‐Docker 등
    - *모니터링·보안 에이전트*, *특수 네트워크 도구* 등에서는 “추가 권한” 요구
3. **“privileged” 모드 한계**
    - `-privileged` 실행 시 모든 보안 레이어 해제 → 컨테이너 탈출·호스트 권한 탈취 위험
    - **예시 탈출**: 호스트 파일시스템 마운트 ▶︎ 사용자 추가 ▶︎ SSH·크론잡 실행 ▶︎ 호스트 완전 장악
4. **권장 해법: “최소권한” 보안 레이어 구성**
    - 컨테이너에 **필요한 권한만** 부여
    - **Capabilities**
        - root 권한을 잘게 쪼개어(cap_net_raw, cap_sys_admin 등) 필요한 것만 허용
        - `-cap-drop ALL --cap-add CAP_NET_RAW` 등으로 최소권한 적용
        - ※ `agprove` 등 도구로 실행 중인 앱을 프로파일링하여 **필요 기능만** 식별 가능
    - **seccomp**
        - 시스템콜 화이트리스트로 허용된 syscall만 통과
        - 기본 Docker 프로필(`docker run --security-opt seccomp=...`)을 복사해 “불필요 syscall 제거”
        - `strace -c` 등으로 앱이 실제 사용하는 syscall만 허용하도록 맞춤화
    - **AppArmor**
        - 파일·네트워크·capability 등 화이트리스트 기반 정책
        - 기본 Docker AppArmor 프로필을 수정·추가하여 앱별 최소권한 적용
        - `docker run --security-opt apparmor=myprofile`
    - **cgroups**
        - CPU·메모리·블록 I/O 등 자원 사용량 제한
        - `-memory=300m --cpus=2` 등으로 과도한 자원 소비 방지
    - **Namespaces**
        - 프로세스·네트워크·마운트·IPC·UTS 등 격리
        - 기본 Docker는 PID/NET/MNT/IPC/UTS 분리
        - 필요 시 “user namespace”도 활성화해 컨테이너 내 root 의 실제 권한을 추가 제한
5. **실전 팁**
    - **앱 패키지에 보안 정책 동봉**: GitHub repo 등에 `seccomp.json`, AppArmor 프로필 등 포함
    - **배포 매니페스트에 명시**: Docker 명령어 예시, Kubernetes `securityContext` 필드로 정책 적용법 문서화
    - **사전 프로파일링→CI 통합**: `agprove`, `strace`, AppArmor 툴 등으로 “필요 최소 syscall/파일·capabilities” 추출
    - **“privileged” 사용은 마지막 수단**: 가능하면 피하고, 꼭 써야 할 때만·짧은 시간만 허용

> 핵심: “--privileged” 대신, Capabilities·seccomp·AppArmor·cgroups·Namespaces를 조합해 앱별 “최소 권한 원칙”을 철저히 적용하면, 컨테이너 탈출·호스트 침해 위험을 크게 줄일 수 있습니다.
>