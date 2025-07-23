# CNCF TAG Network and Cloud Native Network Landscape - Zhonghu Xu, Huawei & Nic Jackson, Hashicorp

다중 선택: KubeCon 2025 EU

## CNCF 태그 네트워크 개요 및 참여 안내

**발표자:** Nick Fisher, Jun Hu, Jun Hushi

---

### 1. TAG(Technical Advisory Group) 개념 & 역할

- **기능:**
    - 네트워크 분야 CNCF 프로젝트 지원·조율
    - 사용자·기여자·TOC 간 갭 해소
    - 프로젝트 성숙도·상호 연동성 촉진
    - 교육 자료·가이드 제공
- **주요 책임:**
    - 신규 프로젝트 스폰서링·가이드 (Sandbox→Incubation→Graduation)
    - 프로젝트 간 관계 정의·문서화
    - 커뮤니티 이벤트·워크숍 운영

---

### 2. TAG Network 산하 프로젝트 현황

- **Graduated:** Envoy, CoreDNS, Cilium, Linkerd
- **Incubating:** gRPC, Emissary (구 Ambassador), Gateway API, CNI
- **Sandbox → Incubating 예비 단계:** K8GB (GSLB), Kuma Mesh, Lockr, Quark, CubeSlice
- **새롭게 온보딩된 Sandbox 프로젝트:**
    - **Lockr:** 멀티클러스터 로드밸런싱
    - **Quark:** kubelet 없이 Node·Pod·VMI 시뮬레이션
    - **CubeSlice:** 클러스터 간 서비스 매쉬 연결·Failover
    - **Semant:** Java 전용 eBPF 서비스메쉬
    - **Connect:** gRPC 호환 HTTP/JSON API 게이트웨이

---

### 3. Quark 기반 CNCF 업스트림 CI 확장성 테스트

- **목표:** 실제 VM 없이 수천 개 Node/VMI 생성 → 컨트롤플레인 부하 측정
- **구성:**
    1. **State CRD**: 대상 리소스(노드/포드/VMI) + 전이할 상태 정의
    2. **Quark Controller**: 주기적 조회 후 상태 전이(Ready, Running 등)
    3. **VMI Impersonation**: KubeVirt 서비스어카운트로 위임 → 웹훅 통과
- **효과:**
    - CI 리소스 절감 → 소규모 클러스터에서 대규모 부하 재현
    - 일일·주간 메트릭 그래프화 → 성능 회귀·개선점 자동 감지

---

### 4. TAG Network 재구성 계획 & 참여 방법

- **“Tag Reboot” 개요:**
    - 기존 8개 TAG → 5개로 통합(Developer Experience, Workloads, Foundation, Infrastructure, Operational Resilience & Security)
    - Networking TAG → Infrastructure TAG 산하에 편입
    - Initiative(연구단위) 신설 → Multicluster, AI Networking 등 집중 연구
- **참여 안내:**
    - GitHub Issues → 토론·프로젝트 제안
    - Bi-weekly 미팅 (Pacific 09:00) → 지식 공유·커뮤니티 구축
    - 멘토링 프로그램 (Underrepresented Groups 대상) → [https://mentorship.cncf.io](https://mentorship.cncf.io/)
    - Slack 채널 “#tag-network”

---

> 문의・참여:
> 
> - GitHub: [https://github.com/cncf/tag-network](https://github.com/cncf/tag-network)
> - 선정 회의록・릴리즈 정보: [https://github.com/cncf/tag-network/tree/main/meetings](https://github.com/cncf/tag-network/tree/main/meetings)
> - Slack: CNCF workspace → #tag-network 채널