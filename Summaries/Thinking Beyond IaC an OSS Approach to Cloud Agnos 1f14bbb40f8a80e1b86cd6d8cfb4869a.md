# Thinking Beyond IaC:an OSS Approach to Cloud Agnostic Infr... - Anmol Sachdeva & Kingsley Madikaegbu

다중 선택: OSS 2024 NA

## 1. 기존 Infra-as-Code(IAC)의 한계

- **DSL·도구 학습 부담**: Terraform·Ansible 등 별도 언어·플랫폼 필수
- **팀 간 조율 병목**: Dev↔Infra↔SRE 복잡한 PR·리뷰, 구성 드리프트
- **프로비저닝 불확실**: 오버/언더 프로비저닝→성능·비용 최적화 어려움

## 2. Infra from Code(IFC) 패러다임

- **“앱 코드만 작성→인프라 자동 생성”**
    - SDK/어노테이션으로 필요한 리소스( API, Bucket, Queue 등) 선언
    - 백엔드(클라우드·k8s) 맞게 자동 프로비저닝
- **장점**:
    - 개발자는 비즈니스 로직에만 집중
    - Serverless·이벤트 드리븐 설계에 최적화

## 3. Nitro 기반 데모 요약

1. **리소스 선언**
    - `api = nitric.api()`
    - `bucket = nitric.bucket('images')`
    - `store = nitric.kv('profiles')`
2. **핸들러 구현**
    - `POST /profiles` → profile 생성
    - `GET /profiles/{id}` → 메타데이터 조회
    - `GET /profiles/{id}/upload` → Signed URL 발급
    - 업로드→Vision API OCR 트리거→프로필 업데이트
3. **Local Simulator**
    - `nitric start` → 브라우저 UI 로 리소스·API·아키텍처 시각화
4. **실제 배포**
    - `nitric up` → Pulumi 호출→Cloud Run, API Gateway, Storage… 자동 생성

## 4. 기대 효과

- **생산성↑**: 앱 코드만 작성 → infra 코드 생략
- **구축 속도↑**: 수분 만에 프로덕션급 배포
- **표준화·드리프트↓**: 선언적 프로비저닝
- **유연성↑**: Serverless·k8s·VM 등 자유 선택

## 5. 도입 시 고려사항

- **IAC 병행**: 네트워크·SRE 요건은 기존 Terraform/Helm 등으로 보완
- **플랫폼 팀 역할**: SDK 확장·정책(Guardrail) 개발
- **스택 구분**: dev/qa/prod별 스택 설정 분리
- **AI 보조**: 코드 분석·설정 제안 AI 도구 결합 검토