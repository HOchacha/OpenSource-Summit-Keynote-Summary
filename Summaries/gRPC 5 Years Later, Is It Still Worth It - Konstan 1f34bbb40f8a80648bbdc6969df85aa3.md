# gRPC: 5 Years Later, Is It Still Worth It? - Konstantin Ostrovsky, Torq.io

다중 선택: KubeCon 2025 EU

**gRPC 도입 및 운영 경험 정리**

1. **회사·서비스 개요**
    - **Torque**: 보안 자동화(No-code) 플랫폼
    - **규모**: Go 기반 마이크로서비스, GCP에서 수백만 자동화 작업 수행
    - **기술 스택**: Go ● gRPC ● PostgreSQL, Redis ● Kubernetes
2. **gRPC 도입 배경**
    - **REST 코드 생성 불만족**: Go 환경의 Swagger/OpenAPI 코드 생성 도구 품질 저조
    - **“표준화된” API 프레임워크**: Google·CNCF 지원, 풍부한 언어별 코드 생성
    - **바이너리 프로토콜**→속도·대역폭 절감, HTTP/2 멀티플렉싱
    - **후방 호환성 보장**: IDL 기반 버전 관리 용이
3. **고려 기준·운영 규칙**
    - **Lint · 자동 검사**: 모든 .proto 파일에 일관된 룰 적용
    - **후방 호환성 체크**: ID 삭제·타입 변경 금지
    - **서비스별 독립 정의**: 메시지 공유 대신 서비스 경계 내 중복 허용→커플링 완화
    - **IDL 확장(extension)**:
        - **권한 제어**: RPC마다 필요한 스코프(scope) 옵션 지정
        - **피처 플래그**: API별 기능 온·오프 토글
4. **개발자 경험(DX) 개선**
    - **공용 Docker 도구 이미지**: 개발자 머신에 컴파일러·플러그인 일괄 제공
    - **`buf` 도입**:
        - 통합 `buf generate` ● 종속 관리(서비스 레지스트리)
        - OpenAPI·gRPC Gateway 대체 가능성 탐색
5. **성과·장단점**
    - **장점**
        - **API 표준화**: 팀 간 일관성 ● 코드리뷰 부담↓
        - **코드 생성 툴 신뢰**: 클라이언트·서버 구현 중복 제거
        - **풍부한 미들웨어 생태계**: 인증·로깅·모니터링·RBAC 편의 제공
    - **단점**
        - **gRPC-Web 한계**: REST-like 캐싱·테크니컬 라이터 UX↑ 추가 프록시 필요
        - **로드밸런싱**: HTTP/2 장기 연결 → k8s 기본 LB 비효율
        - **공용 API 부적합**: 외부 고객용 REST API 병행 필요

---

> 결론: “gRPC는 대규모 마이크로서비스에 강력한 API 표준·생산성 향상을 제공. 부가 툴(buf, gRPC-Web 프록시 등)과 결합해 UX 개선 가능.”
>