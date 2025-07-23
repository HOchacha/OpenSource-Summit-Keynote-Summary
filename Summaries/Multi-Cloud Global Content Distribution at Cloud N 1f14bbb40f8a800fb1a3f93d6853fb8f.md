# Multi-Cloud Global Content Distribution at Cloud Native Speeds - Jiri Kremser & Yury Tsarev

다중 선택: OSS 2024 EU

## 1. 개요

- **프로젝트명**: Kubernetes Global Balancer (KGB)
- **목표**: 오픈소스·클라우드 네이티브·벤더 중립적 글로벌 트래픽 분산 솔루션 제공
- **주요 요구사항**
    - 완전 오픈소스·CNCF 기여 가능
    - 멀티클러스터·멀티클라우드·온프레미스 지원
    - Kubernetes API 기반 선언적 관리
    - 인그레스(Ingress) 등 기존 쿠버네티스 리소스와 통합

## 2. 설계 원칙

- **분산 제어**: 중앙 관리 클러스터 없이 각 클러스터에 컨트롤러 배포 → 단일 장애점 제거
- **DNS 기반 라우팅**:
    - CoreDNS(또는 외부 DNS) 확장으로 배포
    - EDNS Client Subnet 활용해 클라이언트 위치 식별
- **헬스 체크 통합**:
    - In-Cluster Liveness/Readiness Probe 기반 상태 판단 → 외부의 단순 TCP/HTTP 체크 불필요

## 3. 핵심 컴포넌트

1. **kgb-controller**
    - `Gslb` CRD 감시·상태 동기화
    - 클러스터별 DNS 레코드(A/NS/Glue) 관리
2. **kgb-core-dns**
    - 각 클러스터에 배포되는 CoreDNS 플러그인
    - 고객 요청에 맞춰 실시간 RRS(응답 레코드 세트) 동적 생성
3. **external-dns**
    - `DNSEndpoint` CRD로부터 Zone delegation(NS/Glue) 자동화

## 4. 지원 라우팅 전략

- **Failover**: Primary→Secondary 순으로 장애 복구
- **Round-Robin**: (대략) 균등 분산
- **Weighted-RR**: 가중치 기반 분산
- **Geo-Proximity**: 클라이언트 IP↔지리 DB 매핑 후 최적 클러스터 응답

## 5. 동작 흐름 (Geo-Proximity 예시)

1. 클라이언트 DNS 조회 → EDNS Client Subnet 옵션 전송
2. `kgb-core-dns` 가 MMDB(Geo IP DB) 조회하여 해당 IP 대역 위치 판단
3. 설정된 `Gslb` 리소스의 Geo 전략에 따라 최적 클러스터 IP 응답
4. 클라이언트가 선택된 클러스터의 Ingress(또는 서비스)로 접속

## 6. 데모 핵심 포인트

- **멀티 클러스터 구성**: AWS(EU), GCP(US)
- **Failover 시나리오**: EU 클러스터 장애 → GCP(US) 자동 전환
- **Geo-Proximity 시나리오**:
    - EU IP → EU 클러스터
    - US IP → US 클러스터
- **Zone Delegation**: `DNSEndpoint` CRD + `external-dns` 로 NS/Glue 자동 설정

## 7. 프로젝트 현황 및 로드맵

- **CNCF 박스 인큐베이팅 제안 中**
- **주요 사용자**: ABSA(남아공 은행), Millennium BCP(포르투갈 은행) 등
- **로드맵**
    1. **Gateway API** 지원
    2. **Cloud DNS**(NS/Glue) 자동화 (AWS, GCP, Azure)
    3. **v1.0** API 안정화

---

> “KGB는 쿠버네티스 상에서 완전 오픈소스·분산 제어·DNS 기반으로 글로벌 트래픽을 고가용·저지연으로 분산시키는 CNCF-친화적 솔루션입니다.”
>