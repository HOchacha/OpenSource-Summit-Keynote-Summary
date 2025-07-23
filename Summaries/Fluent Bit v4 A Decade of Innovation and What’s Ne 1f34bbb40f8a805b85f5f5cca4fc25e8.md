# Fluent Bit v4: A Decade of Innovation and What’s Next - Leonardo Alminana, Chronosphere

다중 선택: KubeCon 2025 EU

# Fluent Bit 4.0: 현황·신기능·향후 로드맵

## 1. 배경 & 필요성

- **데이터 폭증**:
    - 과거 로그 위주 → 현재 로그·메트릭·트레이스·프로파일 등 다양
    - 저장만 하는 비용 낭비 → 실시간 처리·필터링 중요
- **Fluent Bit 목표**:
    - 가볍고(저메모리·저CPU)
    - 벤더·플랫폼 독립적
    - 방대한 입·출력 플러그인 지원
    - *“수집 → 가공 → 라우팅”**을 최소 비용으로

## 2. 아키텍처·주요 컴포넌트

- **입력(Inputs)**
    - 파일(`tail`), `syslog`, Kubernetes 로그/이벤트, OpenTelemetry, Prometheus, 기존 Fluentd 등
- **프로세서(Processors)**
    - 필터링, 데이터 보강(Enrichment), PII 마스킹, 샘플링, 변환(로그→메트릭) 등
- **조건부 처리(Conditional Processing)**
    - FLB 4.0 신규: **프로세서마다** “조건(expression)” 지정 가능
    - 태그가 아니라 레코드 내용에 따라 필터링/가공
- **스팬 샘플링(Trace Sampling) プロセッサ**
    1. **Head Sampling**: 확률 기반(예: 40% 확률로 샵↑)
    2. **Tail Sampling**: 윈도우(buffer) 안의 전체 스팬을 모은 뒤, 조건(에러 여부 등)에 따른 선택
- **출력(Outputs)**
    - Elastic, Splunk, DataDog, Chronosphere, AWS/GCP/Azure, POSIX 파일 등

## 3. FLB 4.0의 주요 신기능

1. **Trace Sampling Processor**
    - 대량의 트레이스→선별 저장
    - “성공”은 확률↓, “에러”는 100% 보존 조합 가능
2. **Conditional Processing**
    - 입력 단계에서 레코드 내용 기반으로 프로세서/필터 동작 제어
3. **TLS 강화 옵션**
    - 최소·최대 TLS 버전, 허용 Cipher Suite 지정
4. **환경 파일 인젝션**
    - 설정에 `@INCLUDE` 방식으로 외부 파일(비밀·Lua 스크립트) 직접 로드
5. **Zig 언어 플러그인(실험적)**
    - 현재 출력 플러그인만 지원, 향후 Input/Filter/Processor도 지원 예정

## 4. 향후 로드맵

- **언어별 네이티브 SDK/플러그인**
    - Go · Rust · Zig 등으로 “입력·프로세서·필터·출력·커스텀” 완전 지원
- **Parallel Pipelines**
    - 단일 인스턴스 내 다중 파이프라인 처리
- **커스텀 플러그인 생태계 확대**
    - 예) cert-manager 연동으로 단명(短命) TLS 인증서 자동 갱신

## 5. Q&A 하이라이트

- **불필요 데이터 억제**
    - *Trace Sampling* & *Conditional Processing* + 로그 필터링으로 “필요한 것만” 전달
- **Fluentd 대체?**
    - “Fluentd는 유지보수(Maintenance) 모드, 기능 확장보단 Fluent Bit로 신규 기능 개발 중”
- **Lua 스크립트 vs. 플러그인**
    - *빠른 프로토타입*: Lua 스크립트
    - *고성능·재사용*: Go/Rust/Zig 플러그인

---

> 참고:
> 
> 
> – 프로젝트 리더 Eduardo의 보다 심화된 온라인 웨비나(20일 후)
> 
> – 공식 문서·Slack 채널에서 최신 플러그인·샘플 코드를 확인하세요.
>