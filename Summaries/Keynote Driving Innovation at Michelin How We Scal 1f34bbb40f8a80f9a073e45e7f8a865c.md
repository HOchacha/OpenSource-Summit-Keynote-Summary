# Keynote: Driving Innovation at Michelin: How We Scaled Cloud & On-Prem In... G. Quennesson & A. Pons

다중 선택: KubeCon 2025 EU

# Open-Source로 전환한 Michan의 Kubernetes 플랫폼

## 1. 배경 및 동기

- **기존 상황**
    - 2018년부터 전사 Kubernetes 플랫폼 운영
    - 2023년 기준:
        - 62개 클러스터 · 42개 위치(공장별 배포 포함)
        - 450개 애플리케이션 배포 단위
        - 36,000개 Pod 규모
- **전환 요인**
    1. **기존 상용 솔루션 단종**
        - 벤더가 신규 제품으로 전략 변경
        - 기존에서 새 솔루션으로의 마이그레이션 경로 미제공
    2. **오픈소스 전략 강화**
        - 전사에 ‘오픈소스 프로젝트 오피스’ 신설
    3. **내부 기술 역량 강화**
        - ‘Buy’→‘Make’ 전환으로 사내 전문가 육성

## 2. 아키텍처 개요

```
[관리 클러스터] ──▶ [워크로드 클러스터(Azure/GCP/On-Prem)]
      │
      ├─ Argo CD       : GitOps 기반 대규모 배포
      ├─ Cluster API   : 클러스터 생애주기 관리
      ├─ Crossplane    : 인프라 선언적 프로비저닝
      └─ Custom “uhe”  : Cluster CR ↔ Secret 연동

[Git 리포지토리]
  ├─ clusters/      : 클러스터 정의·애드온
  ├─ argocd-apps/   : ApplicationSet 구성
  └─ charts/        : Gatekeeper, Calico, custom-sh 등 Helm 차트

```

- **관리 클러스터**
    - Argo CD·Cluster API·Crossplane·uhe 컴포넌트
- **Git 리포지토리 구조**
    - **clusters/**: 클러스터 인벤토리·애드온 선언
    - **argocd-apps/**: ApplicationSet → 클러스터 라벨 기반 앱 동적 생성
    - **charts/**: 공통 애드온(정책·네트워크·공유 스크립트) 차트
- **워크로드 클러스터**
    - Azure, GCP, On-Prem 포함 다중 인프라

## 3. 주요 성과

1. **플랫폼 운영 비용 ↓44%**
    - 전년 대비 개발·운영 비용 대폭 절감
2. **업그레이드 리드타임 단축**
    - 수개월 → 수주 이내 가능
    - 주니어 엔지니어 투입만으로도 대응
3. **인프라 확장에도 비용 효율 유지**
    - 클러스터·리소스 규모 2배 확대하는 동안도
4. **사내 엔지니어 참여도↑**
    - 오픈소스 기반 개발로 동기 부여·전문성 강화

## 4. 결론 및 참고

- **“Buy→Make” 전환**으로 기술 자립도↑
- **오픈소스 생태계 활용**해 비용 절감·민첩성 제고
- 자세한 내용은 사내 블로그 포스트 참조

---

궁금하신 점이 있으면 말씀해 주세요!