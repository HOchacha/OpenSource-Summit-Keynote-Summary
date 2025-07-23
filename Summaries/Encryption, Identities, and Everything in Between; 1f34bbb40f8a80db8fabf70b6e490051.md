# Encryption, Identities, and Everything in Between; Building Se... Lior Lieberman & Igor Velichkovich

다중 선택: KubeCon 2025 EU

## Kubernetes 네트워크 보안: 정책 → ID 기반 접근제어

---

### 주요 보안사고 요약

1. **디버그 포드 외부노출** → 횡단이동으로 민감정보 유출
2. **Target 해킹 (2013)**: HVAC 계약사, 방화벽 미설정 → 1.1억 PII/CCN 유출
3. **Equifax (2017)**: Apache Struts 취약 → 1억6천만 SSN 등 유출
4. **Marriott (2018)**: Starwood 인수 후 미탐지 유출 전파 → 3.4억 건 유출

> 공통 원인: “취약점 or 너무 넓은 네트워크 접근” + “네트워크 세분화(분리) 부재” → 횡단이동, 더 민감한 리소스 침투
> 

---

## 1. 기존 솔루션: 네트워크 정책

- **NetworkPolicy (v1)**:
    - 네임스페이스 단위 **IP**⇔Pod 연결 제어
    - Allow만 지원(implicit deny), 클러스터 전체 제어값 없음
    - L3/L4만, 외부(e.g. Ingress) 제한 지원 미흡
    - 대규모 클러스터에서 **라벨→IP 동기화** 비용·부정확
- **AdminNetworkPolicy (알파)**:
    - 클러스터 범위 설정 (우선순위↑, Deny/Pass 지원)
    - 클러스터 관리자용, 일부 CNI 구현 필요
    - 정책 중첩·튜닝 복잡도 상승
- **BaselineAdminNetworkPolicy**:
    - Default-Deny 제어, “허용 전” 모든 트래픽 차단
    - 네임스페이스 수준 정책 강제

> 문제:
> 
> 1. 라벨/IP 식별 의존 → 위·변조 가능
> 2. 정책 실행 부하↑, 관리 복잡↑
> 3. 인증·암호화, L7 보안 미지원

---

## 2. 안전한 분리 위한 요구사항

1. **고정·불변 신원(Identity)**
    - IP⇏Identity, Pod 라벨⇏Identity
    - **ServiceAccount**, JWT 등 “변경 불가” 메커니즘
2. **암호화된 전송**
    - mTLS(TLS)로 인증된 상호연결
    - 미암호화 시 인증 불가
3. **유연한 분리·세분화**
    - 네임스페이스/라벨 외 “신원” 기반 정책
    - L3/L4/L7 일관 제어
4. **운영관리·표준화**
    - 인증·권한부여 API 표준
    - 다양한 CNI/Service Mesh 호환성

---

## 3. Identity (신원) 기반 권한부여 제안

- **ServiceAccount** 중심
    - 변경 불가 → Pod 재시작해야 변경
    - Kubectl/(view) 권한으로 임의 변경 차단
- **인증서(VKubelet cap-‘PodCertificate’) 발급**
    - KEP 4307 “Pod Certificates” 활용 → Kubelet이 SA별 cert 발급
    - mTLS 전송 시, SAN ⇒ `spiffe://cluster/ns/sa`
- **AuthorizationPolicy** (예: Istio)
    - `from: principal: spiffe://…` ⇔ `to: services: payments-backend`
    - 네임스페이스 구분・Cluster-wide 적용 가능

---

### 4. 적용 팁

- **Admission Control** (`PodSecurityPolicy`⋯)
    - Pod 라벨/SA 변경 제한 → 정책 우회 방지
- **mTLS 강제화**
    - 암호화 없이 `AuthorizationPolicy`無效
- **로그·시뮬레이션**
    - **NetworkPolicy logging** / OPA 등 외부 인증 연동
    - `policy-assistant` 등 도구로 “what-if” 테스트

---

## 5. 커뮤니티 표준화 로드맵

- **Kubernetes Enhancement**:
    - KEP-4307 “Pod Certificates” → `alpha→beta` 진행中
    - Sig-Network 논의: “Identity vs IP”, “인증서 vs JWT vs SPIFFE”
- **참여 채널**
    - SIG-Network Slack: #network-policy-api
    - GitHub Discussions: kubernetes/enhancements → `network-policy`

> 결론: Pod 라벨/IP 대신, 고정·불변 Identity + mTLS + 정책엔진 결합 → 예측 가능한 네트워크 분리·보안 실현
> 

---

**Q&A & 참여**

- **GitHub**: kubernetes/enhancements#network-policy
- **Slack**: #sig-network #network-policy-api