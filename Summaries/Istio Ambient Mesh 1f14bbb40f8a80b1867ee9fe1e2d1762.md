# Istio Ambient Mesh

다중 선택: OSS 2023 NA

## 1. 배경: 전통적 Service Mesh의 복잡성

1. **Sidecar 패턴**
    - 각 Pod에 Envoy 사이드카 삽입 → “Connecting / Securing / Observing”
    - 대규모 환경에서 **운영·업데이트·비용·지연** 문제 발생
2. **운영 과제**
    - 사이드카 롤아웃 / 업그레이드 과정의 race condition
    - “Job” 워크로드 종료 시 사이드카 남아 리소스 회수 지연
    - 요청당 Envoy 홉 수가 늘어나는 **레이턴시**
    - 과도한 사이드카 오버프로비저닝으로 인프라 비용 상승

---

## 2. Istio Ambient Mesh 개요

- **컨트롤 플레인(istiod)**: 기존과 동일
- **데이터 플레인**
    1. **ZTunnel (per-node)**
        - L4 전용 터널
        - 애플리케이션 → 노드 로컬 ZTunnel → 원격 노드 ZTunnel → 앱
        - 자동 mTLS 설정
    2. **Waypoint Proxy (per-service)**
        - L7 정책 적용 시만 배포
        - HTTP 라우팅·리트라이·Fault-Injection 등
- **Sidecarless**: 기존 Envoy 사이드카 불필요 → 운영·비용·지연 ↓

---

## 3. 작동 원리

1. **Namespace 레이블링**
    
    ```bash
    kubectl label namespace my-ns istio.io/ambient-enabled=true
    
    ```
    

ISTIO-CNI가 각 노드에 ZTunnel 설치

(옵션) L7 정책 필요 서비스에 Waypoint 배포

istiod가 ZTunnel / Waypoint 구성 배포

1. 데모 요약
클러스터 준비

4-node K8s + istioctl install –set profile=ambient

ZTunnel + CNI 플러그인 배포 확인

간단한 서비스 체인

web-api → recommendation → purchase-history (fake-svc)

Ambient 모드 적용

네임스페이스 레이블

curl → ZTunnel 간 암호화 터널링 (L4)

L4 인증 정책

모든 트래픽 차단 → 특정 서비스 허용

(시간에 따라) L7 Waypoint 정책 적용 시연

1. 기대 효과
구분	개선점
비용	풀사이드카 → 경량 ZTunnel (envoy→~ztunnel), 메모리·CPU ↓
운영 간소화	사이드카 롤아웃 불필요 → 네임스페이스 레이블만으로 온보딩／오프보딩
성능	L7 홉 수 감소, ZTunnel 최적화 → 레이턴시 ↓
2. 다음 단계 & 자료
eBook: “Ambient Mesh Handbook” (무료)

워크숍: [solo.io/academy](http://solo.io/academy) → 무료 Hands-On 튜토리얼

블로그: cost & perf. 최적화 사례 → [solo.io/blog](http://solo.io/blog)

Istio 1.16+: Ambient 모드 프로덕션 준비 중