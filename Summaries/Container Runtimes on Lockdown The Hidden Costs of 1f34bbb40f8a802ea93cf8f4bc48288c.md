# Container Runtimes... on Lockdown: The Hidden Costs of Multi... Lewis Denham-Parry & Caleb Woodbine

다중 선택: KubeCon 2025 EU

## 컨테이너 런타임과 격리(Isolation) 요약

### 1. 컨테이너 런타임의 역할

- **정의**: 컨테이너 이미지를 받아 **생성·시작·종료·삭제** 등 관리
- **기술 요소**
    - **Linux 네임스페이스(namespaces)**: PID, 네트워크, 마운트 분리
    - **cgroups**: CPU·메모리 등 자원 제한
    - **SELinux/AppArmor**: 시스템 콜 필터링·접근 제어

---

### 2. “격리” 필요성

1. **너무 많은 정보→필요한 것만 격리**
2. **멀티테넌시(multi-tenancy)**: 한 노드에 여러 사용자·워크로드 공존
3. **잠재적 공격 경로**
    - **노이즈 이웃(noisy neighbor)**: 자원 독식
    - **런타임 취약점**: runc CVE(“leaky vessel”)
    - **특권 컨테이너**: CIS admin 권한 탈취

---

### 3. 주요 런타임 분류 및 특성

| 분류 | 런타임 예시 | 격리 기술 | 장단점 요약 |
| --- | --- | --- | --- |
| **커널 직접 연동** | runc, crun | 네임스페이스 + cgroups | 경량·퍼포먼스 우수, 커널 공격 면역 없음 |
| **유저스페이스 커널** | gVisor | 유저스페이스에서 시스템 콜 시뮬레이션 | 커널 직접 접근 차단, 오버헤드 존재 |
| **VM 기반** | Kata Containers, Firecracker | KVM 기반 마이크로VM | 강력한 격리, 높은 오버헤드·복잡성 |
| **기타** | bubblewrap, Wom | 사용자네임스페이스, 이미지별 VM | 가벼운 격리, 성능·배포 복잡도는 프로젝트별 편차 |

---

### 4. 격리 수준 선택 시 고려사항

1. **보안 요구도**: 민감도(금융·의료 등)에 따라 VM 기반 vs. 네임스페이스 기반
2. **성능·비용**: VM 격리→추가 CPU·메모리·에너지 비용
3. **운영 복잡도**: 설치·업데이트·관리 용이성
4. **하드웨어 제약**: nested virtualization, 특수 HW 필요 여부

---

### 5. 실전 도입 팁

- **런타임 클래스(CRI RuntimeClass)**
    - Kubernetes [`RuntimeClass`](https://kubernetes.io/docs/concepts/containers/runtime-class/) 설정으로 쉽게 전환
- **컨테이너 엔진 설정**
    - containerd/crio 설정 파일에서 기본 런타임 지정
- **판단 기준**
    - “내 워크로드의 격리 요구도는?” → “퍼포먼스·비용 감내 범위는?” → “운영 편의성은?”

---

### 6. 결론

- **격리는 선택이 아니라 설계 요구사항**
- **다양한 런타임 옵션**(runc→gVisor→Kata→…) 중 **내 환경에 맞는 균형**을 찾아야
- **잘 구현된 격리**는 보안을 강화하면서 성능·운영성을 유지하는 키

> 더 자세한 내용·도구(Am I Contained? 등)는 슬라이드(🎥 YouTube) 참고 바랍니다.
>