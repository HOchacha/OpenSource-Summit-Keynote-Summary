# Booting a Linux Kernel in a Higher Privilege Level - Thara Gopinath & Anna Trikalinou, Microsoft

다중 선택: OSS 2024 NA

## 1. 소개

- **발표자**: Analú (Microsoft)
- **프로젝트**: LVBS (Linux Virtualization-Based Security)
- **동기**: 커널이 완전히 손상되어도 보안 중요 데이터·코드(텍스트, Read-Only 데이터, 키 등)는 하이퍼바이저 경계 밖에서 보호

---

## 2. VBS & VTL 개념

1. **VBS (Virtualization-Based Security)**
    - Windows VBS에서 영감 → 커널 내부가 아닌 하이퍼바이저 경계에서 보안 제공
2. **VTL (Virtual Trust Levels)**
    - VTL-0 (게스트 커널·유저) vs. VTL-1 (보안 커널·유저)
    - VTL 간 계층적 권한: 상위 VTL만 하위 VTL 메모리·인터럽트 제어

---

## 3. 전체 아키텍처

```
┌─────────────┐
│  Hypervisor │ ⟵ ring -1
└───┬─────┬───┘
    │     │
┌───▼───┐ ┌─▼──────────┐
│ VTL-1 │ │ VTL-0      │
│(Secure│ │(Guest      │
│ Kernel│ │ Kernel +   │
│ & Apps)│ │ User Space)│
└───────┘ └────────────┘

```

- **메모리 보호**: EPT/S‐LACR/MBA 통해 VTL-1이 지정한 페이지만 VTL-0 접근 허용
- **인터럽트·하이퍼콜 분리**: VTL 1의 우선처리, VTL 0 ↔ VTL 1 전용 하이퍼콜(`vtl_call`)

---

## 4. VTL-1 부트 순서 (싱글 CPU)

1. **메모리 예약** (`bootloader` cmdline “secure_os=addr,size” → `iomem` 등록)
2. **실제 커널 로드**
    - 게스트 initramfs 내에 `vmlinux` (uncompressed) 읽어와서 물리 메모리에 복사
3. **CPU 컨텍스트 설정**
    - Hyper-V `VpSetVpState` API에 CR0/CR3/RIP/etc registers 전달 → VTL-1 진입 준비
4. **VTLS 전환**
    - vtl_call(`VTL_ENTER`) 하이퍼콜 → CPU 0만 VTL-1로 전환
5. **보안 부트로더 실행** (`start.S`)
    - BSS 클리어, boot_params 구성, Linux 커널 진입
6. **커널 초기화**
    - 최소 기능만 남긴 “secure kernel” 인자로 부팅 → init 부족으로 panic → 사용자 initramfs → `ioctl()` 통해 다시 VTL 0 복귀

---

## 5. 멀티플 CPU 초기화

- **hotplug 인프라 활용**:
    1. VTL-1에서 `register_hotplug_cpu()` 호출 → “핫플러그” 방식으로 보안 파티션에 CPU 등록
    2. `wake_secondary_cpu()` 커스텀 API로 VTL 0→VTL 1 전환 + 각 CPU 진입
    3. VTL 1에서 모든 보안 CPU 부팅 완료 후, VTL 0 idle thread에 제어권 복귀

---

## 6. 주요 보안 기능

- **텍스트·Read-Only 섹션 보호**: 하이퍼바이저 EPT로 불변 설정
- **데이터 무결성**: 민감 데이터(키 등)는 VTL 1 메모리만 읽기/실행 가능
- **등록자(레지스터) 보호**: VTL 1 진입 전용 MSR·제어 레지스터 설정
- **인터럽트·시스템콜 가로채기**: 커널 손상 시 자동 VTL 1으로 예외 처리

---

## 7. Secure Boot & 차후 과제

- **Secure Boot**: UEFI + 하이퍼바이저 측 검증 → VTL 1 커널 체인 로드
- **미완 구현**
    1. **Seccomp-like syscall 필터**, 사용자 TEE(T rusted Execution Environment) 지원
    2. **패치/라이브러리 업데이트**: CVE 대응 위한 in-place CSX(시스템 확장) 메커니즘
    3. **성능 최적화**: 부팅·스위치 오버헤드 감소, SMP 런처 간소화

---

## 8. 오픈소스·참여 안내

- **GitHub**: github.com/microsoft/linux-vbs
- **Upstream 목표**: 각종 드라이버·부트로더 패치 → Linux Mainline, Hyper-V upstream
- **문의·컨트리뷰션**: 이 자리 이후에도 편하게 질문·코드 리뷰 환영!