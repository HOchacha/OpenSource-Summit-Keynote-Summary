# Confidential Computing: Why It Has to Be Cloud, and It Has to Be Open - Mike Bursell

다중 선택: OSS 2023 JN

## 1. Confidential Computing 개요

- **정의**:
    - “데이터 사용 중(in-use)” 상태에서의 기밀성·무결성 보호
    - 하드웨어 기반 TEE(Trusted Execution Environment) 활용
    - 암호화·측정(attestation)으로 신뢰 검증
- **데이터 보호 3단계**:
    1. 전송 중(Transit): TLS/IP-Sec
    2. 저장 중(At-Rest): DB 암호화, 파일시스템 암호화
    3. 사용 중(In-Use): Confidential Computing

## 2. Cloud 환경과 신뢰 최소화

- **클라우드 스택 문제**:
    - 호스트(OS·하이퍼바이저)·펌웨어·부트로더 등 다수 벤더·관리 주체 혼재
    - 전통적 가상화·컨테이너만으로는 “호스트→게스트” 공격 차단 불가
- **TEE 솔루션**:
    - 게스트 메모리 페이지 암호화 → RAM→CPU 복호화
    - 각 TEE마다 별도 키 → 상호 간·호스트 접근 차단
    - 칩(SoC)은 최소 신뢰(trust-root), 나머지는 불신

## 3. 오픈 소스·감사 가능성

- **신뢰 관계 최소화**:
    - “나는 CPU만 믿는다” → CSP·OS·하이퍼바이저 불신
- **감사 가능(auditable)**:
    - 부트로더·커널·TEE 런타임 등 핵심 구현 전부 공개
    - 규제·감사 대비·투명성 확보

## 4. Confidential Computing Consortium(CCC)

- **목표**:
    - 오픈 TEE 프레임워크·사양 표준화
    - 생태계 확장(칩·OEM·OS·CSP·ISV·스타트업 협업)
- **주요 프로젝트**:
    - Open Enclave SDK (멀티-TEE 지원)
    - Asylo (Intel SGX 프레임워크)
    - Enarx (WebAssembly + RISC-V TEE)
    - Attestation CA (증명서 발급·검증)
    - SPDM (보안 프로비저닝 프로토콜)
- **참여 방법**:
    - SIG/TAG 가입 → 기술 사양·RFC 기여
    - GitHub PR → 코드·문서 기여
    - 기업 회원 → 솔루션·제품 연계

## 5. 주요 Q&A 요점

- **멀티코어 지원**: 각 코어별 개별 암호화 키 → 동시 접근 차단
- **실물 물리 공격**: Side-channel 등 고난도·장기 물리 제약 필요
- **GPU/가속기**: IETF·산업 워킹그룹서 확장 중
- **워크로드 경계**: 컨테이너 vs. VM vs. TEE 경계 정의 유연

> 핵심 메시지: “신뢰 최소화 + 감사 가능 + 오픈 소스”로 클라우드 데이터 보호 혁신 가능합니다.
>