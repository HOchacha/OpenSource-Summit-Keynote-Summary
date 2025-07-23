# Tutorial: Debugging with Strace - A Peek Behind the Scenes of Linux Processes - Avikam Rozenfeld

다중 선택: OSS 2024 NA

## 1. 개요 & 동기

- **발표자**: Intel HBC 모니터링 엔지니어
- estrace( syscall tracer )를 처음 접하면 “가벨리시(난독문자)”로 느껴지지만, 익히면 디버깅·학습에 필수 도구

## 2. 시스템 콜 & 파일 디스크립터

- **시스템 콜**: 사용자 공간 애플리케이션 ↔ 커널 인터페이스
- **파일 디스크립터**: 열기(open)→번호(fd) 획득→read/write/close 등 모든 I/O에 fd 사용
- 프로세스 생성: fork → execve → exit_group → 부모는 wait( )로 자식 종료 수거
- 스레드는 clone 로, clone(flags) → fork·thread 동시 지원

## 3. estrace 원리

- `strace [-p PID] <command>`: 지정 프로세스+자식의 모든 시스템 콜·시그널 인터셉트
- 기본 출력 → stderr, `o file` → 파일에 저장

## 4. estrace 권장 플래그

```bash
strace \
  -f \         # 자식 프로세스까지 추적
  -y \         # FD→파일·소켓 등 구체적 매핑 출력
  -s 256 \     # 문자열 최대 256바이트까지 출력 (기본 32B)
  -ttt \       # 마이크로초 단위 타임스탬프
  -T \         # 각 콜 소요 시간(ms)
  -o out.log   # 출력을 out.log에 기록

```

## 5. 주요 예제

1. **`cat file`**
    - `strace cat file` → I/O 콜(open/read/write/close)+터미널 출력 뒤섞여 난독
    - `strace -o log cat file` → 로그만 분리 가능
2. **SSH DNS 타임아웃**
    - poll(…) 타임아웃(5 s) → port 53 UDP → `/etc/resolv.conf` 확인 → 첫 NS 불응답
3. **리다이렉션 & 파이프**
    - `strace cmd >out 2>err` → fd 1 → out, fd 2 → err
    - `strace cmd | grep …` → fd 1→파이프, `|&` 는 fd 2 포함
4. **프로세스 생성 흐름**
    - 쉘에서 `ps | sort | less`
    - `bash` → clone()×3 → execve(ps)/execve(sort)/execve(less) → wait() 수집
5. **100% CPU 무한 루프**
    - Python `while True: pass` → 시스템 콜 없이도 CPU 바쁜 루프 추적 가능
6. **Fork–Exec–Wait 상세**
    - fork→(부모↔자식 구분)→ 자식 `execve(…)` → `exit_group()` → 부모 `waitpid()`
7. **DNS 서버 파일 권한**
    - 일반 사용자: `/etc/resolv.conf` 읽기 권한 없음 → DNS 실패
    - root: 동일 명령 성공
8. **`sudo` 추적**
    - setuid 바이너리: 보안상 `strace` 차단
    - 우회: 다른 쉘(PID)에서 root 권한으로 `strace -p $PID`

## 6. 결론

- estrace = “시스템 내부 엑스레이”
- **디버깅** (지연·실패 원인 추적), **학습** (OS·프로세스 흐름 이해)
- 위 플래그 조합으로 모든 호출·시간·FD 매핑을 한번에 확보 가능