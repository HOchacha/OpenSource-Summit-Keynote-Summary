# Tutorial: Linux Memory Management and Containers - Gerlof Langeveld, AT Computing

다중 선택: OSS 2023 EU

아래는 Keller 님의 “Linux 메모리 관리 및 컨테이너” 튜토리얼 강의를 한국어로 요약한 내용입니다.

---

## 1. 세션 개요

1. **리눅스 메모리 관리 기본**
    - **부트 이후 메모리 구조**
        - 커널(고정영역) + 동적 slab 캐시
    - **프로세스 메모리 소비**
        - 코드(Code) · 데이터(Data) · 스택(Stack) · 힙(Heap)
        - demand paging: 실행 시점에 페이지 단위로 로드
    - **페이지 캐시(Page Cache)**
        - 디스크 I/O 가속을 위해 여유메모리를 자동 활용
    - **메모리 부족 시 동작**
        - `kswapd` 동작(Active↔Inactive LRU 페이지)
        - `swappiness`에 따라 PageCache vs. 프로세스 스왑 비율 조정
        - 스왑 공간(full) → OOM(Kill)
2. **실습 도구: `use_mem`**
    - 임의 크기(가상·실제) 메모리 할당·반복(메모리 누수 시뮬)
    - malloc, mmap, System V/Posix 공유메모리 지원

---

## 2. 메모리 관리 심화

- **페이지(Page)** 단위(대개 4 KiB)
- `/proc/[pid]/statm` 혹은 `ps -lo sz,rss`로
    - **VSZ**(virtual size, 페이지 단위)
    - **RSS**(resident size, KB 단위)
- **코드 공유**
    - 실행 파일(공유 라이브러리) 코드는 프로세스 간 1회만 로드
    - 데이터·스택·힙은 프로세스별 별도 로드
- **페이지 교체 구조**
    - **Active↔Inactive** LRU 리스트(익명·파일 캐시 분리)
    - `kswapd`가 Low→High 임계값 사이를 오가며 스캔
- **Swappiness**
    - 0 → PageCache만 축소 → 프로세스 스왑 지양
    - 200 → 프로세스 스왑 적극

---

## 3. Cgroups v2 및 컨테이너 적용

- **Cgroups**로 프로세스 그룹별 메모리 “보장(low)”·“제한(max)”·“스왑합(max+swap)” 설정
    
    ```bash
    # 보장 150 MiB
    echo 150M > memory.low
    # 제한 300 MiB
    echo 300M > memory.max
    # limit+swap 총합 500 MiB
    echo 500M > memory.swap.max
    
    ```
    
- **컨테이너 런타임(e.g. Docker/Podman)**
    - `-memory`, `-memory-swap` → 위 cgroup 파일에 자동 반영
    - 메모리 초과 시 해당 컨테이너 프로세스만 OOM(Kill)
- **쿠버네티스**
    - Pod → 각 컨테이너별 `resources.requests/limits` 지정
    - ▶ QoS 클래스(`Guaranteed`/`Burstable`/`BestEffort`) 결정
    - ▶ `Guaranteed`인 경우 `memory.max`=`limit`과 같아짐 → OOM 방지 우선권

---

### 주요 팁

- **`swappiness` 조정**으로 스왑 vs. 캐시 경향 제어
- **`memory.low`** 로 핵심 프로세스 메모리 보장
- 컨테이너별 `resources.limits.memory` 로 격리된 OOM 방지
- **`/proc/[pid]/oom_score_adj`** 로 OOM 대상 우선순위 조정

---

발표 자료나 데모 `use_mem` 코드가 필요하시면 별도 말씀해 주세요!