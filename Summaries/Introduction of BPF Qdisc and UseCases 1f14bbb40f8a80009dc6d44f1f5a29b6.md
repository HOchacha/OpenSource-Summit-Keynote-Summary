# Introduction of BPF Qdisc and UseCases

다중 선택: OSS 2024 NA

- **발표 대상**: BPF 기반 Qdisc 개발 현황 (RFC v8 패치셋)
- **목표**:
    1. 커널 모듈 작성·배포 없이 사용자 정의 스케줄러(큐 디스크) 구현
    2. 다양한 Qdisc 조합·실험을 빠르고 안전하게 수행
    3. BPF로 네이티브 Qdisc 수준 성능 달성

---

### 1. 설계 기반: `struct_ops`

- 커널 `qdisc_ops` 함수 포인터 구조체를 그대로 BPF 프로그램으로 교체
- **장점**
    - BPF 링크 모델(`bpf_link`) 자동 지원 → attach/detach 간편
    - 향후 ops 확장 시 UAPI 변경 불필요
    - 기존 Qdisc 개발 경험 그대로 활용 가능 (inq/deq 시그니처 동일)

---

### 2. 사용자 구현 범위 축소

- 필수 BPF 콜백:
    1. `enqueue(skb *skb)` → 큐 내부 자료구조에 스케줄 대상 skb 삽입
    2. `dequeue() → skb *` → 큐에서 다음 skb 반환
- 선택적 콜백: `init()`, `reset()`, `peek()` 제공 (인프라 레벨 구현)

---

### 3. BPF 확장 사항 3가지

1. **자동 참조 획득/해제**
    - `enqueue` 첫 인자로 넘어온 `skb`에 대한 참조 자동 획득 → 중복 참조 방지
    - 드롭 시 `bpf_qdisc_skb_drop(skb)` 헬퍼로 안전하게 해제
2. **dequeue 시 참조 반환**
    - `dequeue` 반환된 `skb *`는 소유권 이전(leak) 검증 지원
3. **직접 컬렉션 삽입 기능**
    - BTF 기반 노드 타입 인식 → `BPF_LIST_HEAD` 등 컬렉션에 `skb` 노드 직접 삽입
    - `exclusive ownership` 노드 타입 도입
        - 기존 `bpf_rb_node` 대비 소유자 필드 제거한 `bpf_rb_node_excl`, `bpf_list_node_excl`
        - 메모리 레이아웃 충돌 없이 기존 `struct sk_buff` 내 삽입 보장

---

### 4. 사례 & 성능 평가

1. **FIFO Qdisc (예제 코드)**
    - BPF `LIST`, `spinlock`으로 간단 구현
    - 테스트: loopback 장치에 64바이트 패킷 1M개 전송
    - 성능: 기존 BPF v7 대비 **+7.6%** 처리량 향상 → 네이티브 `pfifo_fast`와 거의 동일
2. **FQ + EDT(rate limit) 연동**
    - EDT: 패킷 크기 기반 타임스탬프 계산 → rate limiting
    - 패킷 손실 정보(`BPF map`) 전달해 지연 보상
    - 결과: 100 Mbps 제한 하, 랜덤 드롭 발생 시 손실 보상 전후 처리량 비교
3. **MQ + 네트워크 에뮬레이터 통합**
    - MQ 큐별 독립 에뮬레이터 → 전역 상태 불일치
    - BPF map으로 상태 머신 중앙 관리 (good/bad state 전이)
    - 이론적 모델 대비 정확도 향상 및 MQ 락 경합 ↓

---

### 5. 정리

- **BPF Qdisc** 인프라는:
    - 커널 모듈 없이 유연·안전한 Qdisc 개발 환경 제공
    - 네이티브 성능 근접, BPF 자체 이점 활용해 교차 컴포넌트 최적화 가능
- **향후 계획**:
    - 추가 ops(예: `sched_classifier`, `tune`) struct_ops 지원
    - 사용자 피드백 기반 API·헬퍼 개선 후 메인라인 커밋 진입 준비

---