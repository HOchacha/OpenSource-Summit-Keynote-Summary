# The State of Prometheus and OpenTelemetry Interoperability - Arthur Sens & Juraj Michálek

다중 선택: KubeCon 2025 EU

## Prometheus & OpenTelemetry 연동 현황 요약

### 발표자

- **Arthur (Grafana Labs)**
    - Prometheus 유지보수자, OpenTelemetry 콜렉터 기여자
- **Uray (SRE, SW3)**
    - Observability 전문가, OpenTelemetry·Prometheus SIG 활동

---

### 1. Pull vs. Push 모델 철학 차이

- **Prometheus:** Pull 기반
    - 장점: 서비스 디스커버리, 실패 스크레이프 감지(`up` 메트릭)
    - 단점: 메트릭을 애플리케이션 메모리에 상주, 변화 없는 값 반복 수집 → 자원 낭비
- **OpenTelemetry:** Push 기반
    - 장점: 메모리 부담 없음, 변화 시에만 전송
    - 단점: 서비스 디스커버리 부재 → “어플리케이션 사라짐 vs. 네트워크 문제” 구분 어려움

---

### 2. 협력으로서의 Prometheus 3.0.0

- **OpenTelemetry 지원 강화를 위해** Prometheus v3.0.0 대규모 개편
    - **UTF-8 라벨** 기본 허용 → 한글·특수문자·도트(‘.’) 등 그대로 사용
    - **OTLP(≒Push) 리시버** 정식 기능화(`-enable-feature=remote-write-receiver`)
    - **메트릭명 변환 전략**(`otel_metric_prefixes` 등) 지원
    - **OpenMetrics 포맷**에 Native 히스토그램·Exemplar 추가 예정
    - **Remote Write V2**
        - Native 히스토그램·Exemplar·메타데이터·생성시간 지원
        - UTF-8·부분 전송(partial write) 통계 제공

---

### 3. OpenTelemetry → Prometheus 전환 과제

1. **Delta 계측 → 커뮤니티컬(cumulative) 변환**
    - OTLP `delta` 자료형을 Prometheus `counter`로 처리하려면
        1. Collector `delta_to_cumulative` 프로세서(State 유지)
        2. Prometheus 내장 프로세서(Collector 없이 단일 프로세스)
    - 향후: 쿼리 시 변환 or 신규 PromQL 함수 검토
2. **리소스 속성(Resource Attributes) 관리**
    - **전략①:** 모든 속성 → 라벨 _(카디널리티/메모리 폭증)₩₩
    - **전략②:** 단일 `target_info` 메트릭 + PromQL `join` _(사용 어려움)₩₩
    - **전략③:** `promote_resource_attributes` 옵션으로 일부만 라벨화
    - **차기 계획:**
        - OpenTelemetry EEP(Entities) 제안 도입 → 컨텍스트 단위 계측 지원
        - Parquet 백엔드 등 고카디널리티 처리 실험

---

### 4. 향후 과제 & 참여 요청

- **Prometheus v3.0.x 시리즈**: OpenTelemetry 네이티브 지원 계속 강화
- **OTEL-Prometheus Spec Sync**: UTF-8·히스토그램·델타 지원 명세 조율
- **커뮤니티 협업**:
    - GitHub 이슈·PR 통해 기능 제안
    - SIG-Metrics, SIG-OpenTelemetry 참여
    - Mentorship 프로그램([https://mentorship.cncf.io](https://mentorship.cncf.io/))

> 문의/기여:
> 
> - Prometheus GitHub: [https://github.com/prometheus/prometheus](https://github.com/prometheus/prometheus)
> - OpenTelemetry Collector GitHub: [https://github.com/open-telemetry/opentelemetry-collector](https://github.com/open-telemetry/opentelemetry-collector)
> - CNCF Mentorship: [https://mentorship.cncf.io](https://mentorship.cncf.io/)