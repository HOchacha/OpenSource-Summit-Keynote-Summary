# Kubernetes Meets Confidential Computing - The Different Ways of Scaling Sensitive... - Moritz Eckert

다중 선택: OSS 2023 EU

## 1. 개요

- **Confidential Computing**: 데이터 사용 중(in-use) 보호
- 하드웨어 기반 **Trusted Execution Environment** (TEE) 활용
- 주요 목표: 메모리 암호화, 원격 증명(Remote Attestation), 코드·데이터 은폐

---

## 2. TEE (Trusted Execution Environment) 모델

### 2.1 Process-level TEE: Intel SGX

- **장점**:
    - 매우 작은 “Enclave”
    - 메모리 페이지 단위 암호화
- **단점**:
    - Context-switch 오버헤드(성능 저하)
    - 시스템 콜 호환성 부족 → 래퍼(Gramine, Enarx 등) 필요

### 2.2 VM-level TEE: AMD SEV / Intel TDX

- VM 전체(Guest OS 포함)를 암호화된 컨텍스트로 분리
- **장점**:
    - “Lift-and-Shift” 방식으로 컨테이너·VM 이식성 개선
    - 컨텍스트 전환 부담↓ (단일 VM 단위)
- **지원 HW**: AMD SEV, Intel TDX, ARM CCA, IBM Secure Exec

### 2.3 GPU TEE

- NVIDIA H100 → GPU 메모리·커널 보호
- PCIe 장치 전반으로 확장 전망

---

## 3. 주요 Use Cases & Threat Model

1. **인프라 보안**
    - “누가” 클라우드 인프라를 통제하느냐(테넌트·서비스 제공자)
2. **다당사자(Multi-Party) 워크플로우**
    - 동형암호 등과 결합해, 민감 연산 신뢰 보장
3. **AI 프라이버시**
    - 모델 소유권·입출력 데이터 보호
4. **소프트웨어 공급망 보안**
    - 빌드·서명 환경 격리 → 최종 실행 환경까지 신뢰 체인

---

## 4. Kubernetes 연동 전략

### 4.1 SGX-to-Container (라이브러리 / 런타임 래퍼)

- **도구**: Gramine, Enarx, Open Enclave, Marble Run
- **단계**
    1. SGX 드라이버(Device Plugin) 설치
    2. 컨테이너 이미지에 SGX 런타임 포함
    3. `Deployment`에 SGX 어노테이션 추가 → 스케줄링
    4. **Marble Run**: Enclave 증명·구성 맵 신뢰 전달, 상호 인증

### 4.2 Confidential Containers (CRI Shim)

- **cncf 프로젝트**: 모든 Pod를 Confidential VM으로 포장
- Pros: Lift-and-Shift 가능
- Cons: Nested VM 어려움 → 원격 하이퍼바이저(윈도우 / Azure Preview) 우회

### 4.3 Constellation (Cluster-wide 보호)

- **패턴**: 컨테이너→VM이 아닌, **노드(VM) 단위**로 TEE 적용
- **장점**:
    - 컨트롤 플레인 포함 가능 → “인프라 vs 운영자” 분리
    - 일반 Kubernetes API 호환
- **사례**: OCCRP(언론 자유 프로젝트), 민감 데이터 시스템

---

## 5. 데모 & 사례

- **SGX 기반 컨테이너**: Gramine + Marble Run
- **Confidential Containers**: `kubectl apply`만으로 Confidential VM Pod
- **Constellation**: `constl apply` → 인증된 Kubernetes 클러스터 배포
- **실제 적용**:
    - 독일 의료정보망(ePA) → SGX 기반 서비스
    - OCCRP → Constellation on AWS

---

## 6. 결론 & 시사점

- **“전체 스택 신뢰 경계”** 축소 → CPU/GPU TEE만 신뢰
- **적용 전략**
    1. SGX Enclave: 신규 앱 / 최소 컨텍스트
    2. Confidential Containers: 컨테이너 단위 보호 (프로토타입)
    3. Constellation: 전체 클러스터 보호 (Lift-and-Shift)
- **차세대 과제**:
    - 모니터링·로깅 (암호화된 컨텍스트 외부 관찰)
    - TEE 간·K8s 내·외부 상호 원격 증명 체계

---

## 7. 참고

- CNCF Confidential Computing SIG: [https://github.com/cncf/sig-confidential-computing](https://github.com/cncf/sig-confidential-computing)
- Marble Run: [https://github.com/confidential-containers/marble-run](https://github.com/confidential-containers/marble-run)
- Confidential Containers: [https://github.com/confidential-containers/confidential-containers](https://github.com/confidential-containers/confidential-containers)
- Constellation: [https://github.com/confidential-containers/constellation](https://github.com/confidential-containers/constellation)
- Gramine (Library OS): [https://github.com/gramineproject/gramine](https://github.com/gramineproject/gramine)
- Enarx: [https://github.com/enarx/enarx](https://github.com/enarx/enarx)