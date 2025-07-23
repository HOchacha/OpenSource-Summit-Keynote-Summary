# A WASM Runtime for FaaS Protected by TEE - Sara Wang & Yongli He, Intel

다중 선택: OSS 2023 EU

---

## 1. 기조: 기밀 컴퓨팅(Confidential Computing)

- **보호 대상**: “데이터 사용 중(in-use)” 단계까지 암호화·격리
- **활용 예**:
    - 금융권 이상 거래 탐지
    - 여러 기업이 데이터 공유 없이 공동 학습하는 연합 학습(Federated Learning)
- **주요 하드웨어 TE(Trusted Execution) 기술**:
    - Intel SGX / upcoming TDX
    - AMD SEV
    - Arm TrustZone / Realms

---

## 2. 전통적 TE 모델

- **TrustZone**: “Secure World / Normal World” 이원화된 커널
- **하이퍼바이저**: 서로 다른 TE VM들을 호스트
- **문제점**:
    - 개발자마다 SDK·툴체인 달라진다
    - 이미지 빌드·디버깅 복잡

---

## 3. 컨테이너 수준의 TE: Confidential Containers

- **목표**: Kubernetes + 일반 이미지 워크플로우 유지하면서 컨테이너 단위로 메모리 암호화
- **구성요소**:
    1. **T-이미지 빌드**: 일반 이미지 → 암호화된 레이어 추가
    2. **레지스트리**: 암호화 이미지 저장·서빙
    3. **KBS(Key Broker Service)**: 디스크 복호화 키 등 민감 정보 관리·전달
    4. **CRI 런타임 확장**: shim → Intel SGX 드라이버 넣고 `rocky` 샘플처럼 Enclave 실행
- **장점**: TE를 컨테이너 수준으로 표준화 · K8s 친화적

---

## 4. WebAssembly + TE 를 합치는 이유

1. **신속 기동**: 서버리스(FaaS) 같은 환경에서 “초경량 런타임 + 스냅샷”
2. **언어 독립 모듈화**: WASM 모듈은 플랫폼·언어 불문
3. **워크플로우 통일**:
    - **Knative Functions**: 코드를 WASM으로 컴파일 → TE VM 내에서 실행
    - **Spin**: “WASM 네이티브 클라우드” 플랫폼 → Spin 바이너리만 Enclave에 띄우면 서비스
4. **개발자 경험 개선**:
    - 통상적인 FaaS/K8s 오퍼레이터 도구를 그대로 이용
    - TE 개발 복잡도 ↓, 기존 코드·CI 파이프라인 활용

---

## 5. TE 내 WASM 런타임 예시

- **ego**:
    - Intel SGX/TDX 등에서 추가 코드 수정 없이 “wasmer” 등 런타임만 설치 후
    - `ego build` / `ego run` 으로 바로 Enclave 내 WASM 구동
- **WAMR(Wasm Micro Runtime)**:
    - 임베디드·엣지·클라우드용 초경량(~백 KB) WASM 엔진
    - TE 파티션 없이도 Enclave 바이너리에 링크 가능

---

### 맺음말

> WASM + 컨테이너 TE 조합을 통해
> 
> - **기밀 컴퓨팅**(메모리·코드·데이터 암호화)
> - **초경량 서버리스**(즉각 스냅샷·스케일 투 제로)
> - **언어·플랫폼 독립성**
> - **기존 DevOps·GitOps 워크플로우 재활용**
>     
>     이 모두를 달성할 수 있음을 보여 주었습니다.
>     

발표 자료와 코드 예제들은 각 오픈소스 리포지터리에 게시되어 있으니, 관심 있는 플랫폼/런타임으로 직접 시도해 보시길 권합니다!