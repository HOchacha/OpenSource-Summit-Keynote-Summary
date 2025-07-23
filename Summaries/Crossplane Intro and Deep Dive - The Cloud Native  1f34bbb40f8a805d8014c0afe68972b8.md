# Crossplane Intro and Deep Dive - The Cloud Native Control Plane Framework - Jared Watts & Nic Cope

다중 선택: KubeCon 2025 EU

# Crossplane v2 발표 요약

## 1. Crossplane란?

- **Cloud-Native Control Plane**
    
    Kubernetes 위에서 동작하며, 컨테이너 외 모든 클라우드·온프레 자원을 Kubernetes API로 관리
    
- **Managed Resource**
    
    AWS S3, GCP Storage 등 외부 리소스를 Kubernetes CRD로 선언
    
- **Composition**
    
    여러 Managed Resource를 조합해 사용자·개발자에게 편리한 상위 추상화(Custom API) 제공
    

## 2. v1 vs. v2 주요 변경 사항

| 구분 | v1 | v2 |
| --- | --- | --- |
| Composite Resource | Cluster-scoped | Namespaced / Cluster-scoped |
| Managed Resource | Cluster-scoped | Namespaced / Cluster-scoped |
| Claims | Namespace→Cluster CR 자동 생성 (Claims) | Claims 제거 → 직접 CR 생성 |
| Crossplane 설정 필드 | spec 최상위 | spec.crossplane 하위 |
| API 호환성 | – | v1→v2 상위호환 (legacy scoped 지원) |

## 3. v2 주요 기능

1. **네임스페이스 지원**
    - XRD(CompositeResourceDefinition)·ManagedResource 모두 namespaced 가능
    - 테넌시 분리, 권한 관리, GitOps 활용성↑
2. **간결한 API**
    - 기존 Claims/Cluster-XR→생략
    - kubectl get 만으로 연관 리소스 일괄 조회
3. **언어 선택 자유로운 Composition Function**
    - Helm-style YAML, Python, KCL, Go 등 사용 가능
    - 커뮤니티 제공 함수(templating, IAM policy 생성 등) 재사용
4. **점진적 마이그레이션 보장**
    - 모든 v1 clusters-scoped 리소스 v2에서 그대로 지원

## 4. v2 데모 흐름

1. XRD 생성: `kubectl apply -f app.xrd.yaml`
2. Function 설치: `kubectl crossplane install function composite-kcl:v0.1.0`
3. Composition 등록: `kubectl apply -f app.kcl.composition.yaml`
4. App CR 생성 후 배포 확인:
    - `kubectl apply -f my-app.yaml`
    - `kubectl get app,mydeployment,myservice`

## 5. 향후 로드맵 & 기여

- **곧 릴리스(1.20)**
    - Composition ChangeLog, 실시간 Composition(watch 기반)
    - 관찰성·메트릭 강화
- **커뮤니티 기여**
    - GitHub: `crossplane/crossplane` → CONTRIBUTING.md 참조
    - Slack: #crossplane 채널 문의
- **참고 URL**
    - 공식 웹사이트: [https://crossplane.io](https://crossplane.io/)
    - v2 Preview Docs: [https://doc.crossplane.io/v2-preview/](https://doc.crossplane.io/v2-preview/)