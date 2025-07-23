# Simplifying the Networking and Secur... Bill Mulligan, Anna Kapuścińska, Bowei Du & Amir Kheirkhahan

다중 선택: KubeCon 2025 EU

---

## 1. 발표 개요

- **발표자**
    - Thomas Graf (SELinux·eBPF 창시자)
    - Amir Khan (DB Schenker 플랫폼 엔지니어)
    - Bowie (Tetragon 유지보수자)
- **세일리움 프로젝트 범위**
    - **네트워킹(CNI)**: 레이어3부터 레이어7까지, 클러스터 메시·BGP 게이트웨이·프록시 대체 등
    - **관측(Hubble)**: 서비스 간 트래픽 가시화·디버깅
    - **보안(Tetragon)**: eBPF 기반 정책으로 투명 암호화·네트워크 정책·커널 이벤트 후킹

---

## 2. Cilium 1.17 주요 기능

- **네트워크 QoS**: 보장·버스트·베스트이포트 트래픽 클래스 지원
- **멀티클러스터 서비스 API**: 글로벌 서비스 일관 처리
- **Gateway API v1.2**: 표준 게이트웨이 지원
- **네트워크 정책 성능**: 우선순위·검증 기능 추가
- **운영·스케일**:
    - BGP 피어링 메트릭·레이트 리미팅
    - 일별·주별 스케일 테스트 개선

---

## 3. DB Schenker 사례 (Amir Khan)

- **환경**: AWS 위 셀프 관리 GKE, Kafka 운영, 노드 ‘소몰이(cattle)’ 방식 패치/교체
- **마이그레이션**:
    1. 단계적 전환(“near zero–downtime”)
    2. Hubble 메트릭에 Kubernetes 메타데이터 추가
    3. **eBPF 호스트 라우팅**: 네임스페이스 탈출 후 고속 패킷 처리
    4. **투명 암호화**: Helm 플래그 한 줄로 mTLS 자동 구성·키 롤링
    5. **CNI → eBPF 라우팅**: pod→pod 간 단순 IP 테이블 → eBPF 라우팅 전환
- **향후 계획**:
    - 대체 HTTP 게이트웨이 구축
    - **클러스터 메시** 기반 멀티클러스터 pod→pod 보안 통신

---

## 4. GKE 스케일링 (Bill)

- **GKE ‘Data Plane v2’**: eBPF 기반 세일리움 CNI로 네트워킹·보안·관측 제공
- **65 000 노드 실험**:
    - 초당 500 pod churn 대응
    - Kafka mirroring도 무리 없이 운영
- **스케일 과제**:
    - Kubernetes Control Plane 부하(노드·Pod churn)
    - 전체 기능 중 고스펙용 ‘스케일링 프로필’ 제안
    - mega-cluster(수만 노드) 네트워크 정책 설계

---

## 5. Tetragon 업데이트 (Anna)

- **eBPF Go 라이브러리**: Windows 초도 지원 → 향후 Windows eBPF production ready
- **tetragon debug**:
    - BPF 프로그램별 실행 횟수·CPU 사용량 측정
    - BPF 맵별 메모리 사용량 측정
    - Prometheus 메트릭 자동 노출
- **신기능**:
    1. 구조체 필드 단위 검사 지원(`struct_ops`)
    2. 관찰·강제 모드 전환 API
    3. CEL 필터: JSON 이벤트 고급 필터링

---

## 6. 커뮤니티 & 로드맵

- **자격인증**: “Cilium Certified Associate” 시험 개설 → Golden Kubeastronaut 자격 포함
- **연례 보고서**: “The Year of Kubernetes Networking” 공개(확장성·기능 성장)
- **SIG 신설**: SIG Scalability 등 특화 개발그룹 가동
- **멘토링 프로그램**: GSoC·LFX 등에서 Helm·Operator·UI 리팩토링 기여
- **Roadmap**:
    - ClickHouse 공식 백엔드 지원
    - 실시간 토폴로지·메트릭 오버레이
    - UI 현대화(통합 시각화 라이브러리)

---

**문의 및 참여**

- GitHub: github.com/cilium/cilium
- CNCF Slack: #cilium, #tetragon
- 매주 수요일 SIG 미팅(유럽 시간 오후), 월 2회 APAC 세션
- 사례연구 제안: 귀사의 프로덕션 도입 스토리를 공유하세요