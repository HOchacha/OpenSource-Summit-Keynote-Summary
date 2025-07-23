# Reinventing Container Linux for the Wasm Era (and More) with System Extensions - Andrew Randall

다중 선택: OSS 2024 NA

## 1. 소개

- **발표자**: Microsoft Azure Core Linux 팀 PM, Andy
- **주제**: systemd 기반 “시스템 확장(System Extensions)”을 이용해 Flatcar Container Linux(이하 Flatcar)를 모듈화·컴포저블하게 개조하는 방법

---

## 2. Linux Distro 종류

1. **General-Purpose Linux**
    - 수천 개의 패키지, 가변 파일시스템(apt/yum 등)
    - 장점: 거의 모든 워크로드 대응 가능
    - 단점: 대규모 취약점·업데이트 노출, 개별 머신마다 설정·패키지 편차(스노플레이크)
2. **Container-Optimized (Special-Purpose) Linux**
    - 불변(immutable) 루트 파일시스템, 최소 패키지, 컨테이너 런타임만 제공
    - 장점: 공격면 최소화, 선언적 재현 가능한 배포
    - 단점: 추가 패키지·도구 설치 불편

---

## 3. “Best of Both Worlds”: 모듈식 확장

- **목표**: 최소 OS 특성 유지하면서도, 필요한 기능은 선언적 “시스템 확장(CSX)”으로 쉽게 추가
- **아이디어**:
    1. Immutable OS 이미지
    2. “확장 레이어”로 추가 파일만 묶어 디스크 이미지(CSX)로 배포
    3. 부팅 시 overlay → 최종 조합된 파일시스템 완성
    4. 업데이트 역시 CSX 단위로 독립 관리

---

## 4. systemd System Extension 개요

- **CSX(Bundle)**:
    - 단순 파일 모음 → 불변 OS 위에 overlay 적용
    - `systemd-csexec` 이용해 런타임·부팅 시 마운트/로드
- **업데이트**:
    1. 독립 CSX: HTTP(S)로 주기적(타이머) 폴링·설치 → `systemd-cis-update`
    2. OS 연동 CSX: Flatcar의 기존 OS 업데이트 프로토콜 확장
    3. OS 내장 CSX: 기존 OS 이미지에 바로 포함

---

## 5. Flatcar 적용 사례

1. **컨테이너 런타임 분리**
    - `containerd` → CSX로 포장 → Podman/Docker 등 다른 런타임 교체 가능
2. **플랫폼 에이전트(OEM tools)**
    - Azure Linux Agent, VMware Tools 등 → CSX로 배포 → 인플레이스 업데이트 지원
3. **Cluster API용 K8s 제어면**
    - 별도 CSX로 관리 → OS & K8s Control-Plane 독립적 버전업

---

## 6. WebAssembly 전용 Dro 구상

- **왜 WASM?**:
    - 모든 아키텍처 공통 바이너리, 빠른 시작·보안 샌드박스, 엣지/서버리스에 적합
- **WASM-Optimized Linux**:
    - 기존 CSX 구조 그대로 → 컨테이너 런타임 대신 WASM 런타임·툴만 올림
    - Docker/Docker-대신 `wasmtime`, `wasmcloud` 등으로 경량화

---

## 7. CSX 제작·배포 워크플로우

1. **CSX 정의** (`flatcar-csx-bakery`):
    - 포함할 파일·메타데이터 작성 → `bake` 스크립트로 `.raw` 이미지 생성
2. **서버에 배포**:
    - 체크섬 + CSX 이미지 → HTTP(S) 저장소(OCI 레지스트리, GitHub Releases 등)
3. **Ignition 설정**:
    
    ```yaml
    storage:
      files:
      - path: /etc/cis-update-WMTIME.toml
        contents: { source: "https://…/WMTIME-update.toml" }
    systemd:
      units:
      - name: cis-update.timer
        enable: true
      - name: cis-update.service
        enable: true
    
    ```
    
4. **오프라인 배포**:
    - `bake --embed`로 Flatcar OS 이미지에 CSX 번들 포함

---

## 8. 커뮤니티·다음 단계

- **관련 그룹**
    - Linux UAPI WG (LInux userspace API)
    - CNCF Special-Purpose OS SIG
- **Flatcar 참여**
    - GitHub: github.com/flatcar
    - 월간 오피스아워, 이슈·PR 환영
- **향후 과제**
    - CSX UX 개선(Dockerfile 유사 문법, CLI 통합)
    - 업데이트 롤아웃 정책 강화(Canary, 배치, 롤백)
    - OStree·rpm-ostree 비교·통합 방안 고도화

> “시스템 확장(System Extensions)”을 통해 불변 OS의 안전성·일관성은 유지하고, 필요 기능은 모듈처럼 손쉽게 추가·업데이트하세요!
>