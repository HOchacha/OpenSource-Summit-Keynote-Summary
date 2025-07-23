# How We Moved Spotify To a Proxyless gRPC Service Mesh - Erik Lindblad & Erica Manno, Spotify

다중 선택: KubeCon 2025 EU

### Spotify의 DNS 기반 서비스 디스커버리 한계

- 기존 시스템(“Nameless”):
    - Kubernetes EndpointSlice + VM 하트비트 → Cloud DNS에 SRV 레코드로 노출
    - 문제1: 응답 크기 한계(65 KB) 초과 → 대형 서비스 디스커버리 실패
    - 문제2: DNS 캐시로 인한 TTL 불확실성 → 서비스 언레지스터 지연, 장애 복구 지연
    - 문제3: 단방향(DNS→클라이언트)·L7 라우팅 불가 → 세부 라우팅·실시간 트래픽 제어 불가능

---

### 대안 검토 (2013→2024)

1. **GCP Traffic Director** (매니지드 XDS)
    - 장점: TD 전용 기능 풍부
    - 단점: 자사 요건(Proxyless gRPC, 계층적 라우팅) 미지원, 확장성 제한
2. **Istio(Envoy) Mesh** (Self-hosted or 매니지드)
    - 장점: XDS 지원·L7 강력 라우팅
    - 단점: 사이드카 오버헤드, 프록시 관리 복잡
3. **Nameless 확장(XDS 직접 구현)** ← 선택
    - 장점: 기존 데이터·설계 재활용, 직접 최적화 가능, 가벼운 구현(XDS ADS/ADS만)
    - 단점: 자체 프로토콜 해석·유지보수 필요

---

### 아키텍처 리팩터링 & XDS 도입

1. **Nameless 리팩터링**
    - Nameless Discovery·Spanner·Cloud DS → elim → `shameless`(K8s CRD 인메모리) + DNS 리졸버
2. **XDS 직접 구현**
    - ADS(Aggr. Discovery Service) 기반 XDS만 구현 → Proxyless gRPC 지원
3. **프레임워크 연동**
    - Java 관리 채널(Apollo): DNS↔XDS 채널 이중화 + “XDS 사용 여부” DNS TXT 플래그토글
    - 서비스 메타데이터로 “XDS eligibility” 판정

---

### 점진적 도입 전략 & 기능 활용

- **“% rollout” + 그룹별 on-boarding** (수백 개 서비스 → 전 사로 확장)
- **핵심 기능**
    1. **트래픽 스플릿**: 임퍼러티브 API로 가상 라우팅 규칙 삽입
    2. **영역별 로컬리티 LB**: Zone-aware + 클라이언트 부하보고 기반 라우팅
    3. **서비스 의존성 그래프**: 전역 연결·트래픽 흐름 시각화
- **Rollback “패닉 버튼”**: DNS 플래그 토글 → 3–4분 내 DNS로 전환, 0 ~ 미미한 영향

---

### 결과 & 향후 과제

- **성과**:
    - DNS 불확실성 해소, 응답 지연 대폭 감소
    - 새로운 라우팅·제어 요구 충족
    - 점진적·무중단 롤아웃 가능
- **과제**:
    - 초기 복잡도·버그 대응, 교육 필요
    - 다양한 프로토콜·다양 언어 지원 확대

> 결론: “직접 구현한 XDS + 점진적 전환” 전략으로, DNS 한계를 극복하고 Spotify 전역 서비스 디스커버리에 필요한 고급 제어 기능을 안정적으로 확보했습니다.
>