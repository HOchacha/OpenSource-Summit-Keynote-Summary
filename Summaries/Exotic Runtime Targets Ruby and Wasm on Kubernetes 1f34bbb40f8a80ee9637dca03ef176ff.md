# Exotic Runtime Targets: Ruby and Wasm on Kubernetes and GitOps Delivery Pipelines - Kingdon Barrett

다중 선택: OSS 2023 EU

아래는 Kingdon Barrett 님의 “Exotic Runtimes: Kubernetes + Ruby + Wasm” 강연 내용을 한국어로 요약한 것입니다.

---

## 1. 발표자 소개

- **Kingdon Barrett**: Weaveworks 오픈소스 지원 엔지니어, Flux 유지관리자
- 유튜브 채널 운영(라이브 코딩, Kubernetes/Flux/DXO 관련)
- 매주 ‘Flux Bug Scrub’ 세션 진행

---

## 2. 오늘의 주제

- **“Exotic Runtimes”**: Kubernetes 위에서 Ruby 코드 + WebAssembly(Wasm) 모듈을 함께 쓰는 사례
- **핵심 키워드**: Kubernetes · GitOps · Ruby · Wasm · Kubernetes Operator

---

## 3. Why Wasm?

1. **안전한 샌드박스**: `wasmtime`·`wasmer` 등 런타임이 메모리·시스템 호출을 격리
2. **이식성**: 플랫폼·언어 독립 바이너리(.wasm)
3. **경량화**: 빠른 시작·소규모 페이로드(단, Ruby 인터프리터 자체를 Wasm으로 빌드하면 30MB→비실용적)
4. **WASI**(WebAssembly System Interface)로 파일·네트워크 접근 지원

---

## 4. Ruby + Wasm 도전기

- **목표**: GitHub Container Registry에서 Flux 패키지 다운로드 수를 스크래핑→OCR→시계열 수집
- **웹페이지 파싱**: HTML → 정규식 대신 CSS 셀렉터 사용
- **러스트( Rust )로 Wasm 모듈 작성**
    - `cargo + wasm32-wasi` → `strip` → `wasm-opt`
    - Ruby 런타임(`wasmer-ruby` gem)에서 호출 → 숫자·타임스탬프 반환

---

## 5. Kubernetes Operator in Ruby

1. **kubectl + GitOps**(Flux) 기반 클러스터 준비(예: EKS + Flux)
2. **Ruby ‘kubeclient’-gem** 사용해 오퍼레이터 구현
    - 단 몇 줄(`upsert`·`delete` 핸들러)로 CRD 리콘실러 작성
    - **Finalizer** 관리, **Server-Side Apply**, **Readiness Await** 지원
3. **CRD 설계**
    - `Project` CRD → FluxCD 설치 단위
    - `Leaf` CRD → 각 패키지(Flux CLI, Source Controller 등)별 다운로드 수 보관
    - `.status.downloadCount`, `.status.lastUpdate` 필드

---

## 6. 데모 흐름

1. Rust로 만든 Wasm 모듈 빌드 (`make build`)
2. Ruby 스크립트(`wasmer`)로 `.wasm` 호출 → 페이지 파싱 후 다운로드 수 추출
3. Kubernetes에 Operator 배포 → `Project` CR 생성
4. Operator가 `Leaf` 리소스 생성 & 상태 갱신 → `.status.downloadCount`에 숫자 기록

---

## 7. 얻을 수 있는 이점

- **모듈성**: 페이지 파싱 로직을 Wasm 모듈로 분리
- **언어 독립**: Rust·Ruby 등 혼합 사용 가능
- **안전성**: Wasm 샌드박스로 외부 코드 격리
- **GitOps 통합**: Flux 통해 배포·상태 감시 자동화

---

## 8. 부가 참고

- **Spin**: Wasm 기반 서버리스 프레임워크(로컬 테스트용)
- **WASI/WAGI**: 표준 입출력 기반 웹서버 서빙
- **kustomize/Helm**: Flux 연동 차트 예제(헬름 차트 간소화 라이브러리 ‘helmet’ 사용)
- **추가 링크**
    - GitHub: ~~/stats-tracker 폐쇄형 repo(Operator 코드)
    - kubeclient-ruby: [https://github.com/kontena/kubeclient](https://github.com/kontena/kubeclient)

---

Wasm + Ruby + Kubernetes 조합은 아직 ‘틈새’이지만, **안전한 샌드박스**, **모듈화**, **GitOps 자동화**라는 장점이 있습니다.

관심 있으시면 발표자료·코드를 직접 살펴보시고, Flux Bug Scrub 세션에서 질문 주세요!