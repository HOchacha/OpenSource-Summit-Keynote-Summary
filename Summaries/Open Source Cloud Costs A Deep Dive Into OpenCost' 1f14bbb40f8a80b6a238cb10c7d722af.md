# Open Source Cloud Costs: A Deep Dive Into OpenCost's Impact on Enterprise...- Matt Ray & Don O'Neill

다중 선택: OSS 2024 NA

## 1. 발표자 및 소개

- **발표자**: Matt Ray (OpenCost 커뮤니티 매니저) & Don O’Neal (Microsoft/Salesforce SRE·DevOps)
- **세션 주제**: 오픈소스 기반 클라우드·Kubernetes 비용 모니터링 프로젝트 “OpenCost”

## 2. OpenCost 프로젝트 개요

- **라이선스**: Apache 2.0, CNCF 호스팅
- **기원**: 기존 CubeCost 사양(spec) + 구현체를 2022년 6월 CNCF 기여
- **범위 확장**: Kubernetes → 일반 클라우드(IaaS) 비용 → SaaS 비용 → 탄소 배출 비용

## 3. 사양(OCI Cost Spec) 주요 개념

- **Kubernetes 리소스** (pods, namespaces, deployments…) 별 비용 분담 모델
- **공유 자원** (클러스터 관리 요금, Load Balancer 등) 분할 방식
- **집계 단위**: CPU·메모리 요청·리미트·실제 사용량, 스토리지, 네트워크 등

## 4. 모니터링 대상 및 연동

1. **Kubernetes 비용**
    - EKS/GKE/AKS 관리 요금 + EC2/GCE/VM 인스턴스 비용
    - PV, LoadBalancer, Network Egress 등
2. **Cloud 비용**
    - AWS Cost & Usage Report, Azure 및 GCP 빌링 데이터 처리
3. **탄소 배출 비용**
    - Cloud Carbon Footprint 프로젝트 연동, 노드·파드 단위 탄소량 측정

## 5. Microsoft (MuSoft) 도입 사례

- **환경**: AWS EKS 5개 클러스터, 약 1,000명 개발자
- **목표**: 서비스 팀별 정확한 비용·효율성 파악 → 차세대 요금 배분·최적화
- **비교 평가**: CubeCost(유료), CloudHealth(제한적) → OpenCost 선택

## 6. 구축 아키텍처

1. **클러스터 내 OpenCost 배포**
    - `opencost` 네임스페이스, Prometheus 종속
2. **데이터 파이프라인**
    - Prometheus 스크레이핑 → OpenCost API → CSV 일일 내보내기 → S3 버킷
    - 사내 데이터 레이크 적재 및 BI 툴로 분석·대시보드 생성

## 7. 운영 중 얻은 교훈

- **Prometheus 필요**: 반드시 클러스터에 Prometheus 설치
- **장기 저장소**: Prometheus는 단기 분석 용도, 장기 보관은 외부 시계열 DB 사용 권장
- **할인율 반영**: OpenCost는 온디멘드 요율 기반, 사후 요금 조정(Reconciliation)은 별도 처리

## 8. 확장 및 팁

- **Cost Plugins**: 1.10부터 SaaS·추가 비용원(예: Datadog) 플러그인 지원
- **데이터 소스**: CSV(기본 일 1회), API, Parquet, Athena 연동
- **UI·도구**:
    - 기본 웹 UI (React)
    - Grafana 대시보드 리포지토리
    - Backstage 플러그인(개발 중)
    - CLI(`kubectl cost`)

## 9. 요약

OpenCost는 완전 오픈소스 비용 모니터링 플랫폼으로, Kubernetes뿐 아니라 전체 클라우드·SaaS·탄소 비용까지 투명하게 추적·분석할 수 있게 해 줍니다. 다양한 배포 옵션(Docker, Helm, SaaS), 플러그인 아키텍처, 그리고 Prometheus 기반 데이터 수집 파이프라인을 통해 조직별 맞춤 비용 최적화 워크플로우를 지원합니다.