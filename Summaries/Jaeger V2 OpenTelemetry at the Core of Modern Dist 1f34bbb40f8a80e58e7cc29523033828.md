# Jaeger V2: OpenTelemetry at the Core of Modern Distributed Tracing - Jonah Kowall, Paessler

다중 선택: KubeCon 2025 EU

**발표 제목**

Jæger를 활용한 분산 트레이싱의 진화 — Jonah Cowell

**발표자**

Jonah Cowell (Jæger 프로젝트 메인테이너)

---

## 1. 분산 트레이싱의 필요성

- **마이크로서비스 디버깅**: 어디(Which 서비스)에서 병목이 발생했는지, 누구(Which 팀)에 이슈를 할당할지 정확히 파악
- **의존성 매핑**: 서비스 간 호출 관계를 그래프 형태로 시각화 → 전체 토폴로지 이해
- **장애 원인 분석**: 단일 트랜잭션(Trace)을 따라가며 오류·지연 지점 파악
- **운영 모니터링**: 에러율·응답 시간·처리량 등의 SLA 지표 추출

---

## 2. Jæger 개요 및 주요 기능

1. **Trace·Span·Tag 모델**
    - **Trace**: 엔드투엔드 요청
    - **Span**: Trace 내 개별 호출 구간
    - **Tag**: 사용자 지정 메타데이터(팀, 환경 등)
2. **UI 주요 뷰**
    - **타임라인 뷰**: 시계열로 각 Span 중첩 표시
    - **크리티컬 패스**: 전체 지연에 가장 기여하는 호출 구간 강조
    - **토폴로지 뷰**: 서비스 호출 트리 형태 시각화
    - **로그 연계**: Span별 로그·메트릭을 함께 조회
3. **Hot Rod 데모**
    - Uber 모사 “Hot Rod” 애플리케이션
    - 프런트엔드 → DB, Redis 호출 패턴 실시간 Trace
    - “계단 현상” 발견→병렬 처리 제안, 데이터베이스 락 문제 진단

---

## 3. 모니터링 지표 자동 생성 (Span Metrics)

- **SpanMetrics Processor** (OpenTelemetry)
    - Trace 데이터를 받아 **요청수(Rate)**, **에러율(Error Rate)**, **지연(Duration)** 메트릭으로 변환
    - Prometheus 스크랩·원격 기록(Remote Write) 지원 → 대시보드·알람 활용
- **Jæger UI 연동**:
    - “Monitoring” 탭에서 실시간 메트릭 차트 조회
    - Trace·Metrics 데이터 간 상호 드릴다운 가능

---

## 4. Jæger v2 – 리플랫폼·백엔드 통합

1. **OpenTelemetry 기반 재구성**
    - Collector·SDK 전면 OTEL 이관
    - 기존 Jæger 프로토콜·저장소(Cassandra·ElasticSearch·OpenSearch) 호환 유지
2. **단일 바이너리화**
    - Collector·Agent·UI 모두 하나의 실행 파일(`jaeger-all-in-one`)로 배포
    - YAML 설정파일로 구성 관리
3. **확장 기능**
    - **Kafka 큐잉**: 대용량 Trace 유입 시 뒷단 안정적 배치 처리
    - **원격 샘플링(Remote Sampling)**: 애플리케이션 재시작 없이 샘플링율 동적 변경
    - **Helm 차트·Operator 제공**: 쿠버네티스 배포 간소화

---

## 5. 로드맵 & 향후 과제

- **ClickHouse 공식 지원**: 대규모 트레이스 저장소로 CI·QA 완료 중
- **UI 리팩토링**: 현대적 시각화 라이브러리 도입, 일관된 UX 제공
- **실시간 토폴로지**: OpenSearch 기반 뷰→Spark 워크플로우 의존 제거
- **추가 백엔드**: Cloud Storage, NewSQL 등 대체 스토리지 실험

---

**자료 & 참여**

- GitHub: [https://github.com/jaegertracing/jaeger](https://github.com/jaegertracing/jaeger)
- CNCF Slack/#jaeger, CubeCon 부스 또는 발표 후 Q&A 환영