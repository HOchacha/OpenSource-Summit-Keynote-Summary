# Securing Connections: Defending Telco Workloads in the Cloud Era - Phil Porras, Accuknox

다중 선택: OSS 2024 NA

## 1. 배경 & 동기

- 5G · 소프트웨어 정의 네트워크(SDN) → 모든 RAN/CN 기능이 가상화된 워크로드로 구현
- **미션 크리티컬**(군사·헬스·산업 로보·차량 등) 환경에선 “한 번의 보안 사고”가 치명적
- **Private 5G** vs **Telecom 5G** vs **공공 5G** (차량·로보·Next-Gen 911…)
- 운영자·통합사업자·Early adopter 모두 “보안 보증” → 배포·확장에 필수

---

## 2. 보안 요구 영역

1. **에지(Edge) 보안**
    - 단말 ↔ gNB(기지국) L3 프로토콜 침투 탐지
    - SDR 공격(패킷 주입·가로채기·교란)
    - **SEC-RAN**: CU/DU 내 eBPF 기반 보안 텔레메트리 삽입
    - **X-Apps** / **R-Apps**:
        - 실시간 이벤트 기반 탐지(Expert systems)
        - 비실시간·대량분석(ML / DL)
2. **제어면(Control-Plane) 보안**
    - NFV 워크로드 최소권한(Zero-Trust)
    - 배포→오케스트레이션 단계별 컴플라이언스(3GPP·MITRE·NIST·DoD 권고)
    - **의도 기반 보안**: “의도(Intent)” → 보안 정책 → 런타임 적용
    - 보안 엔진(OPA, kube-armor, Falco 등) 연동

---

## 3. 오픈소스 툴·프로젝트

- **mobile-flow**
    - RAN L3 메시지·통계 → e2-compliant 보안 텔레메트리
- **5G Spectre**
    - Expert System X-App: 실시간 셀룰러 공격(DoS·프로토콜 위반 등) 탐지
- **deepwatch**
    - LSTM / Transformer 기반 비실시간 이상 패턴 학습·탐지
- **kube-armor**
    - eBPF / LSM: 컨테이너·VM·로우메탈 최소권한 런타임 방어
- **Nimbus (ONF NEF-SIG)**
    - 보안 의도 → 어댑터 → 엔진별 정책 매핑·자동화

---

## 4. 주요 데모·사례

1. **SEC-RAN** — Oran CU/DU에 보안 텔레메트리 모듈 삽입
2. **mobile-flow + 5G Spectre** — 가상화된 5G 시험망(Coliseum)에서 공격 시뮬→실시간 알림
3. **deepwatch** — 대규모 프로토콜 캡처에 ML 모델 학습·예측
4. **kube-armor + NEFIO/Nimbus** — 제어면 NFV 최소권한 자동화 워크플로우

---

## 5. 향후 과제 & 전망

- **의도 기반 보안 자동화**: 자연어 → 추상 보안 의도 → 정책·엔진 매핑
- **AI Ops 6G**: 자동 스펙트럼·리소스·보안 관리
- **하이브리드·이종망 통합**: 유·무선·클라우드 공통 보안 프레임워크
- **오픈표준 강화·컴플라이언스 툴체인**