# Service Mesh Tracing Observability with Kuma and OpenTelemetry - Wenhan Shi, Kong Inc

다중 선택: OSS 2023 JN

## 1. Kuma 서비스 메쉬 개요

- **프로젝트**: CNCF Sandbox, Kong에서 개발
- **아키텍처**
    - Control Plane
    - Data Plane (각 서비스마다 Envoy sidecar)
- **주요 기능**
    - 서비스 간 라우팅·로드밸런싱
    - 제로 트러스트 네트워킹 (TrafficPermission)
    - 인증·인가(mTLS, JWT 등)
    - 로깅·모니터링, 레이트 리미팅
    - Fault Injection, Traffic Tracing

## 2. Kuma 배포 모델

- **Standalone**
    - 단일 쿠버네티스 클러스터 내 Control Plane + sidecar
- **Multi-Zone**
    - Global Control Plane + Zone별 Control Plane
    - Zone 간 자동 페일오버(HA)
    - Kubernetes·VM·베어메탈 혼합 환경 지원

## 3. Policy(정책) 관리

- **CRD 기반**: 서비스 코드 변경 없이 YAML 한 줄로 정책 선언
- **예시: TrafficPermission**
    
    ```yaml
    apiVersion: kuma.io/v1alpha1
    kind: TrafficPermission
    metadata:
      name: allow-demo-to-reg
    spec:
      sources:
      - match:
          kuma.io/service: demo
      destinations:
      - match:
          kuma.io/service: reg-server
    
    ```
    
- **제공 항목**: 인증·인가, mTLS, ACL, RateLimit, FaultInjection, TrafficTrace 등 20여 가지

## 4. OpenTelemetry 개요

- **역할**: 벤더 중립 텔레메트리(Trace/Metric/Log) 수집·전송 프레임워크
- **흐름**
    1. API/SDK로 계측 → 애플리케이션에서 Trace/Metric/Log 생성
    2. OTLP 프로토콜로 Collector 전송
    3. Collector → 다양한 Backend(Honeycomb, Jaeger, Elasticsearch 등) 전송
- **장점**: 자동·수동 계측, 멀티 Backend 연동, 표준화된 데이터 포맷

## 5. Kuma + OpenTelemetry 통합

- **MeshTrace CRD**: 메쉬 단위로 OTLP Collector 연결
    
    ```yaml
    apiVersion: kuma.io/v1alpha1
    kind: MeshTrace
    metadata:
      name: default
    spec:
      backends:
      - name: otel-collector
        type: open_telemetry
        conf:
          endpoint: otel-collector.kuma-system.svc:4317
          headers:
            x-api-key: <HONEYCOMB_API_KEY>
    
    ```
    
- **자동 계측**: Envoy sidecar가 애플리케이션 트래픽 자동 수집
- **Collector 활용**: OTLP 수집 → 필터·처리 후 다중 Backend로 분기 전송

## 6. 데모 흐름 요약

1. `kubectl label namespace mesh-for-devs kuma.io/sidecar-injection=enabled`
2. 데모 앱(Deployment+Service) 배포 → Envoy sidecar 자동 주입
3. Kubernetes Ingress로 외부 트래픽 노출
4. MeshTrace CRD 생성 → OTLP Collector(endpoints + API key) 지정
5. 샘플 요청 1초 간격 전송 → Honeycomb 대시보드에서 Trace 확인
    - `demo → reg-server` 경로별 지연 시간(250ms×4) 및 전체 응답(1s) 시각화

> 결론:
> 
> - Kuma로 서비스 메쉬 계층에서 일관된 보안·로그·Tracing 정책을 적용
> - OpenTelemetry Collector 연동으로 복잡한 분산 시스템의 병목·에러 구간을 손쉽게 가시화 가능