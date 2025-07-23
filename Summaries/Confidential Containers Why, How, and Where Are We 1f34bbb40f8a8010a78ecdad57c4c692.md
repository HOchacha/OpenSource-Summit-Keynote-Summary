# Confidential Containers: Why, How, and Where Are We? - Magnus Kulke, Microsoft

다중 선택: OSS 2023 EU

---

## 1. 기밀 컴퓨팅(Confidential Computing)이란?

- **신뢰 실행 환경(Trusted Execution Environment, TEE)**
    
    CPU 차원의 하드웨어 지원(Enclave)을 이용해, 메모리·CPU 레지스터·캐시까지 암호화·격리
    
- **목표**
    - **데이터 기밀성**: 계산 중인 데이터가 유출되지 않도록
    - **데이터 무결성**: 실행 중인 코드·데이터가 변조되지 않도록
- **주요 사용 사례**
    - GDPR 등 규제 준수를 위한 민감 정보 처리
    - 대형 AI 모델(LLM)을 블랙박스로 고객에 제공
    - 다자간 연합 분석(Multi-Party Computation)
    - 전자교통·이동성 서비스 이용 데이터 안전 가공 등

## 2. 기밀 컴퓨팅의 핵심 개념

1. **Data-In-Use 보호**
    
    – 데이터 “사용 중”에 메모리 등에서 노출되지 않도록 CPU 수준에서 암호화/격리
    
2. **연쇄 측정(Chain of Trust)**
    
    – 부트로더 → 커널 → 초기 RAMdisk → 애플리케이션까지, 각 단계 해시값을 “확장 측정”해 기록
    
3. **원격 증명(Remote Attestation)**
    - TEE 하드웨어가 “보고서(report)”를 생성 → 서비스 제공자(Vendor)ㆍ검증기(Verifier)에 전달
    - 검증기: 서명·측정값 비교→신뢰성 확인 → 비밀키·시크릿을 안전히 전달

## 3. 기밀 컨테이너(Confidential Containers) 프로젝트

- **문제**: 일반 컨테이너는 호스트 커널·하이퍼바이저 관리자에게 메모리 노출
- **해결 방향**:
    1. **가벼운 VM 격리** (예: Kata Containers)
    2. **원격 하이퍼바이저** (클라우드 API로 CVM 생성 → Nested VM 불필요)
    3. **Kubernetes 통합**:
        - `RuntimeClass: kata-remote` → 각 Pod를 CVM 안 TEE로 실행
        - **Cloud-API Adapter**: Kubernetes 오퍼레이터가 CVM 수명 주기 관리
        - **Attestation Agent**: 게스트 내부에서 측정값 수집·원격 증명 요청
        - **Key Broker Service (KBS)**: 증명 성공 시 시크릿/이미지 복호화 키 발급

## 4. 데모·실습 흐름

1. **Docker→Confidential VM**: `RuntimeClass: kata-remote` Pod 스케줄 → CVM 생성
2. **원격 증명**: Agent가 보고서 요청 → KBS·Verifier 통해 검증 → 복호화 키 수령
3. **암호화 이미지 배포**: TEE 안에서 컨테이너 이미지 디스크·메모리 완전 암호화 해제

## 5. 남은 과제

- **이미지 풀링**: CVM으로 대용량 이미지 안전 전송·공유
- **노드·컨트롤 플레인 권한 최소화**: 클러스터 관리자 권한이 Enclave를 건들지 않도록
- **Dynamic Config**: ConfigMap 등 실시간 업데이트 시 TEE 무결성 보장
- **측정·성능 튜닝**: 대량 배포 시 원격 증명 오버헤드·암호화 해제 비용 최소화

---

컨테이너 환경에 “이식성 높은 TEE”를 더해 **“설치 변경 최소화”**로 기밀 컴퓨팅 도입을 쉽게 하자는 것이 핵심입니다.