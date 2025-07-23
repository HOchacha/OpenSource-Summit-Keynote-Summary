# Mind the Gap Between the Future and the Present - Taylor Thomas, Cosmonic

다중 선택: OSS 2024 NA

## 1. 발표 배경

- **“Mind the Gap”**: 런던 지하철 승강장과 열차 사이 경고문 → 현재(컨테이너/K8s)와 미래(Wasm+Components) 사이 간극
- **발표자 이력**:
    - WASM Cloud 핵심 유지·관리자
    - Helm 3 주요 기여자 (––`-wait` 개발자)
    - Kubernetes(AKS 등)·WebAssembly 전도사

---

## 2. WebAssembly(Wasm) 생태계

1. **전용 런타임**: Wasmtime, WAMR, WasmEdge
2. **플러그인/확장**: Envoy, Redis UDF, PostgreSQL 등
3. **서버리스/FaaS**: Fastly, Cloudflare Workers, Deno Deploy
4. **K8s 통합**:
    - **Wrapped**: `containerd` shim (Spin, Krustlet)
    - **Alongside**: k8s + sidecar(예: WASM Cloud Operator)
5. **앱 플랫폼**: WasmCloud, wasmWorkerServer, etc.

---

## 3. 컨테이너 vs. Wasm

- **컨테이너**:
    - 크기: 수백 MB (OS+라이브러리)
    - 언어·플랫폼 종속 (각 아키텍처별 이미지 필요)
    - 보안: cgroup/네임스페이스 기반
- **Wasm**:
    - 크기: 수십 KB~단일 MB
    - 언어·플랫폼 무관, 한 번 빌드→아키텍처 무관 실행
    - Capability–based sandbox
    - 즉시 기동, “콜드 스타트” 거의 無

---

## 4. Wasm **Components** 모델

- **WIT** (Wasm Interface Types):
    - 인터페이스 정의 → `string`, `list<T>`, `enum`, `error` 등
- **장점**:
    1. 언어·플랫폼 불문, “Interface First” 개발
    2. **SDK–for–Free**: 기존 SDK(Go, Python…) → Interface 로직 재사용
    3. **Virtual Platform Layering**: 서비스 체이닝 & 세분화된 권한 주입
- **비교**:
    - Unix pipeline → 단방향·문자열
    - Components → 양방향·구조화·타입 안전

---

## 5. Kubernetes 확장 vs. Wasm

- **K8s 확장 포인트** (7가지) → 실전선택지: CRD + Operator
- **문제**: K8s→“무조건 컨테이너” 페그인홀
- **대안1. Wrapped** (`containerd` shim)
    - ▶ 장점: 네이티브 K8s 통합, ‘Pod’처럼 동작
    - ▶ 단점: 런타임 교체·노드 이미지 변경 필요, K8s 제약 이어받음
- **대안2. Alongside** (WASM Cloud)
    - ▶ 장점: CRD만 설치, 인-Cluster “Host” 런처, 멀티 클러스터·분산 앱
    - ▶ 단점: 새로운 “Component” 패러다임 학습 필요

---

## 6. 라이브 데모

1. **Wrapped**: Spin Cube + RuntimeClass → Wasm 모듈을 `kubectl apply`
2. **Alongside**: WasmCloud Operator + Host →
    - `wash host start` → 로컬·GCP·AKS에 Host 배포
    - `kubectl apply -f wm-distributed.yaml` → 3개 클러스터에 Component 배포
    - `curl` → 분산 Key–Value 카운터

---

## 7. 결론 & 권고

- **“Present vs Future”**: K8s 만 고수→미래 혁신 잠재력 잠금
- **브릿지 전략**:
    1. **Wrapped**: 빠른 PoC, 기존 K8s 파이프라인 재활용
    2. **Alongside**: 점진적 도입, Component + 멀티 클러스터·서비싱 레벨 확장
- **핵심**: WASM 은 “단순 대체 컨테이너” 아님 →
    - **Infra primitives** vs **App–platform primitives**
    - 미래의 분산·안전·폴리시 제어 → Components