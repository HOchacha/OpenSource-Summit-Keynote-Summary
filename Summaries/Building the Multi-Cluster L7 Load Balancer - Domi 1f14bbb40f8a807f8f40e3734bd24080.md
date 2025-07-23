# Building the Multi-Cluster L7 Load Balancer - Dominic Kim & Shawn Jang, NAVER

다중 선택: OSS 2023 NA

## 1. 배경 & 목표

- **Naver**
    - 한국 10대 IT기업·글로벌 커머스(포시마크), 핀테크·콘텐츠·클라우드 서비스
    - 8개 리전·45개 AZ에 58개 K8s 클러스터 운영
- **목표**
    1. IDC(데이터센터) 장애 시 자동 차단·Fail-over
    2. HTTP(L7) 기반 라우팅·서킷브레이킹·Rate-limit
    3. 실시간·API 기반 설정 변경(동기⇄비동기 갭 해소)
    4. 지역·AZ 기준 “가장 가까운” 백엔드 라우팅
    5. 설정·모델 변경 시 무중단 데이터 마이그레이션

---

## 2. 아키텍처 개요

1. **2-Tier NLB**
    - **Tier-1 “Entry”**: 공용 IP 집중 → 여러 클러스터 포워딩
    - **Tier-2 “User-svc”**: 서비스별 Ingress → Istio/Envoy로 L7 처리
2. **헬스체크 서비스**
    - 전(全) 클러스터 간 풀메시 검증 → 비정상 AZ의 DNS 레코드 자동 삭제
3. **DNS 연동**
    - Hidden-Primary → Secondary (AXFR/NOTIFY, 10s 배치)
    - Dynamic DNS로 Fail-over·AZ 차단

---

## 3. 핵심 솔루션

### 3.1 IDC 장애 감지 & Fail-over

- **Health Check**:
    - 각 Gateway ↔ 풀메시 검증
    - 실패 AZ의 A-레코드 자동 삭제 → 클라이언트 차단

### 3.2 L7 프로토콜 지원

- **Envoy/Istio** 사용
- 서비스·도메인·백엔드 모델로 추상화
- UI/Config API 통해 “가상 서비스(VS) + Gateway” 자동 생성

### 3.3 선언적 API & Reconciliation Loop

- **Producer/Consumer 패턴**
    1. API 서버(Producer): “Desired State” 저장
    2. 각 클러스터(Consumer): loop 돌며 실제 K8s 객체 생성/수정/삭제
- **Idempotence** 보장 → sync API로 강제 재실행 가능

### 3.4 글로벌 트래픽 로컬리티

- **GSLB + CNAME** 기반 지역별 DNS 응답
- **Endpoint Priority**
    - 각 백엔드에 리전/존 우선순위 지정
    - 로컬이 불가 시 페일오버 자동 전환

### 3.5 데이터 마이그레이션

1. **Model 변경** → Migration API로 일괄 업데이트
2. **Operation 변경** → sync API 호출로 loop 재실행
3. **Model 간 Dependency** → “trigger event” API로 연결

---

## 4. 정리

- IDC 장애·L7 기능·API 동기화·글로벌 지역화·무중단 마이그레이션
- 모든 과제 → **선언적 모델 + Loop(Reconcile) + Envoy 기반 L7** 으로 해결
- **“Cloud-native L7 LB”** 설계/운영 성공 사례