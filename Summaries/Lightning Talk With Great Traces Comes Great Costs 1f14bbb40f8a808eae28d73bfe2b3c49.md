# Lightning Talk: With Great Traces Comes Great Costs: How to Reduce That Bill? -Prashansa Kulshrestha

다중 선택: OSS 2024 EU

## Distributed Tracing의 비용 이슈와 해법 여정

### 1. 분산 추적(Distributed Tracing) 개요

- **목적**: 요청이 여러 마이크로서비스·인프라·DB를 거치며 동작하는 전체 흐름을 “Trace”로 기록·시각화
- **Sampling 필요성**
    - 모든 Span(추적 단위)을 저장·전송하면 방대한 데이터→높은 저장·네트워크 비용 발생
    - **Head-based**: 요청 시작 시 “로또” 방식으로 샘플링 → 중요 이벤트 놓칠 수 있음
    - **Tail-based**: 요청 종료 시 샘플링 → 모든 정보 파악 가능, 구현·운영 복잡도↑

---

### 2. APM 서비스 이용 시 비용 문제

- 요청량 증가 → 추적 데이터 폭증
- 외부 APM 업체에 네트워크 전송·저장·처리 비용 지불
- “서비스 확장 = APM 매출 증가” 구조 → SRE/플랫폼팀에 부담 전가

---

### 3. 초기 시도 & 실패 사례

1. **혼합 샘플링 모드**
    - 핵심 서비스: Tail-based, 기타 서비스: Head-based
    - 🔥 문제: 서로 다른 샘플링 정책→Trace 단절·맹점 발생 → 팀 간 책임 공방
2. **APM 제공 Drop-rule 사용**
    - Span 속성(중복·불필요 필드) 필터링
    - 🔥 문제: APM UI가 내부 필드를 전제로 설계→도구 오류·렌더링 실패

---

### 4. 전략 전환: 전 서비스 Tail-based + 필터링

- 모든 서비스에 Tail-based 샘플링 적용
- “latency > 100 ms” 또는 “5xx 에러” Trace만 보관
- 🔥 문제: “관측” 아닌 “이벤트 수집” 수준으로 전락, 여전히 막대한 네트워크 전송 비용

---

### 5. 해법 #1: OpenTelemetry 커스텀 샘플러

- **JS 사용자 정의 Sampling 로직**
    - `span.kind === "middleware"` 등 “중간 처리형 Span” Drop
    - `span.attributes.skip === true` 같은 비관심 함수 필터링
- **효과**
    - 예시: 단일 서비스 10 → 6 Spans (–40%)
    - 연결된 3서비스 아키텍처 87 → 53 Spans (–39%)

---

### 6. 해법 #2: OTEL Collector + Tail Sampling Processor

- **Collector**: 서비스⇄APM Provider 사이에 중계자 역할
- **Tail Sampling Rules** (내장)
    - Latency 기반, 에러(5xx) 기반 등
- **이점**
    - APM 전송 전 “후처리”로 네트워크·저장 비용 대폭 절감

---

### 7. 결론 & 교훈

- **목표**: “시스템 인사이트” ⇔ “금전적 파산”은 별개
- **최종 구조**:
    1. 서비스 내부에 OpenTelemetry SDK + Custom Sampler
    2. OTEL Collector 에서 Tail Sampling (Processor)
    3. 필요한 Trace만 APM Provider로 전송
- **핵심**:
    - **사용자 정의 샘플링**으로 불필요한 데이터 대폭 제거
    - **Collector 기반 후처리**로 외부 전송량 최소화

> “분산 추적은 필수이지만, ‘모든 데이터’가 아니라 ‘의미 있는 데이터’만 골라야 비용 효용이 극대화됩니다.”
>