# Tutorial: Developing Portable eBPF Applications Workshop - Lin Sun, solo.io

다중 선택: OSS 2023 EU

---

## 1️⃣ LAB 1 · OOM‐킬러(eBPF) 앱 빌드 & 실행

1. BCC 예제 `oomkill.c`와 Bumblebee 최적화 버전(`steps/oomkill/oomkill.c`) 비교
2. Rim 버퍼로 바꾼 코드를 OCI 이미지로 빌드·푸시
    
    ```bash
    b build -t localhost:5000/oomkill:1 ./steps/oomkill
    b push localhost:5000/oomkill:1
    
    ```
    
3. 로컬에서 단독 실행
    
    ```bash
    b run localhost:5000/oomkill:1
    
    ```
    
4. 쿠버네티스 DaemonSet으로 배포 후 Prometheus → `oomkill_events_total` 확인

---

## 2️⃣ LAB 2 · `opensnoop` eBPF 프로그램 디버깅

1. BCC 예제 `opensnoop.c` vs. Bumblebee 버전(`steps/opensnoop/opensnoop.c`) 비교
2. `bpf_printk()` 디버그 출력문 삽입(→ PID 제대로 읽히도록 shift 32)
3. 다시 빌드·푸시
    
    ```bash
    b build -t localhost:5000/opensnoop:1 ./steps/opensnoop
    b push localhost:5000/opensnoop:1
    
    ```
    
4. 로컬 실행하며 PID 필터링 확인
    
    ```bash
    b run -f 12345 localhost:5000/opensnoop:1
    
    ```
    

---

## 3️⃣ LAB 3 · eBPF 훅 두 개를 한 이미지에 담아 배포

1. `exit_and_oom.c` 파일 살펴보기
2. 두 개의 Rim 버퍼(Exit + OOM)와 두 개 훅(LSM exit + kprobe oom_kill) 결합
3. 단일 이미지로 빌드·푸시
    
    ```bash
    b build -t localhost:5000/exit_oom:1 ./steps/exit_and_oom
    b push localhost:5000/exit_oom:1
    
    ```
    
4. 로컬 & K8s DaemonSet 실행 → `exit_events_total` + `oomkill_events_total` 모두 수집 확인

---

### 다음 단계

- **고급 실습**: `bpf_ringbuf`·`libbpf` 헬퍼 API 사용해 보기 (Lab 4)
- **무료 인증**:
    1. [bit.ly/develop-ebpf-apps-quiz] 접속
    2. 시험 합격 시 “EBPF Apps” 뱃지 획득
- **커뮤니티 참여**: Slack `#bumblebee` 채널(@solo-io) 질문·토론
- **소스 & 문서**: [https://bumblebee.io](https://bumblebee.io/) • [https://github.com/solo-io/bumblebee](https://github.com/solo-io/bumblebee)

남은 컨퍼런스 시간도 즐겁게 보내세요!