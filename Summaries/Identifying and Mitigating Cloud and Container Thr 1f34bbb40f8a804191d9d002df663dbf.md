# Identifying and Mitigating Cloud and Container Threats - Pablo Musa, Sysdig

다중 선택: OSS 2023 EU

## 1. 전체 아젠다

1. 이미지 단계 보안
2. 레지스트리 단계 보안
3. 런타임 단계 보안·Falco 데모
4. 마무리·권장 실천 과제

---

## 2. 이미지 단계

- **소스코드 → 빌드 → 레지스트리 → 런타임** 전 과정을 모두 위협 벡터로 간주
- **최소(슬림) 베이스 이미지 사용**
    - 예: Node.js 표준 이미지는 취약점 수십~수백 개↑, Alpine 계열은 수개↓
- **의존 패키지 최소화**
    - curl, tar, vim, npm 등 공격자가 활용할 수 있는 툴 제거
- **빌드 머신 보안**
    - CI/CD 권한 최소화·암호화·스크립트 주입 방지
    - 빌드 파이프라인 위 암호화폐 채굴 악성코드 방어

---

## 3. 레지스트리 단계

- **접근 제어 & Least Privilege**
- **비공개 Registry → 서명된 이미지**
    - 개발→QA→운영 단계별로 이미지에 디지털 서명
    - 서명 검증(예: Cosign) 통해 무결성·출처 보장
- **취약점 스캔**
    - 이미지 레이어별 취약점 위치 파악 → 문제 레이어만 교체·패치

---

## 4. 런타임 단계 & Falco

1. **왜 런타임 보호가 필요한가?**
    - 컨테이너 탈출, 제로데이, 쿠버네티스 오브젝트 변조, 자격증명 탈취 등
2. **Falco 개요**
    - CNCF Incubating 프로젝트
    - **eBPF 트레이스(또는 커널 모듈) → Ring Buffer → 사용자공간 Falco 프로세스**
    - 시그니처(룰) 매칭 → 이상행위 탐지 → Falco-Sidekick 연계(슬랙·PagerDuty 등)
3. **데모: Log4Shell 익스플로잇**
    - 악성 페이로드 전송 → JNDI LDAP 요청 유도 → 역방향 셸 획득
    - Falco 기본 정책으로 “원격 셸 실행”, “신규 바이너리 생성”, “권한 상승 시도” 등 모두 탐지
4. **주요 룰 예시**
    - `container_shell_event` (컨테이너 내부 쉘 실행)
    - `write_below_root` (루트 이하 디스크 변경)
    - `drop_privs` (권한 이양)
    - `high_fidelity exec_crate` (새 바이너리 실행)

---

## 5. 결론 및 권장 실천 과제

- **이미지 최소화 → 의존성 최소화 → 서명 + 스캔**
- **여러 단계(소스→빌드→레지스트리→런타임) 통합 보안**
- **런타임 모니터링(eBPF/Falco)으로 이상행위 탐지**
- **Least-Privilege, Zero-Trust, Shift-Left** 문화 정착

> 참고자료
> 
> - Falco 공식문서·룰 레퍼런스
> - Falco Sidekick (알림·시각화)
> - “Practical Cloud Native Security” 워크숍 (hands-on)