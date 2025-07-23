# Expanding eBPF’s Reach: From Batteries-Included Auto-Instrumentation To E2E Observab... Dom Del Nano

다중 선택: KubeCon 2025 EU

**“eBPF 확장 활용” 요약 (Dom Donano, Pixie/Cosmic CEO)**

---

### 1. 배경: 관찰성의 진화

- **모놀리식→마이크로서비스/K8s** 전환으로 “세 가지 핵심 관찰성(로그·메트릭·트레이스)” 중요도↑
- **수작업 계측 한계**: 언어·프레임워크마다 직접 삽입해야 해 비용·노력 급증

### 2. eBPF 도입 이유

- **넓은 커버리지**: 커널 수준 훅으로 애플리케이션·컨테이너·SaaS까지 “언어·프레임워크 무관” 전체 시스템 관찰
- **낮은 계측 비용**: 커널 하나만 지원하면, 다수 서비스 자동 모니터링 가능
- **다양성 수용**: 수작업 계측보다 도구 종속성↑(자동), but 유연성 필요

### 3. eBPF 툴 유용성 3대 축

1. **데이터 보강(Data Enrichment)**
    - 커널 이벤트에 컨테이너·K8s 메타데이터(파드·네임스페이스 등) 결합
2. **구조화 API(Structured API)**
    - gRPC·HTTP API 등 명확한 인터페이스로 외부 시스템·UI·CLI 연동
3. **도메인별 확장성(Programmability)**
    - 보안 (→Pulsar)·관찰성 (→Inspector Gadget, Pixie, Hubble) 도메인 맞춤 스크립팅·정책 표현력

### 4. 주요 eBPF 프로젝트 비교

| 툴 | 용도 | 보강 | API | 확장성 |
| --- | --- | --- | --- | --- |
| **InspectorGadget** | 관찰성 | `define`·`include` 자동 컨테이너 매핑 | 클러스터 단위 CLI·API | “가젯”별 모듈 형식, 설치·조합 자유 |
| **Pulsar** | 런타임 보안 | 커널 이벤트 + 프로세스 메타데이터 | Rule 엔진 API | SIS콜·파일·네트워크 추적→정책 DSL로 위협 탐지 & 알림 |
| **Pixie** | 관찰성 | K8s 메타 + 프로토콜 추적 | “PixieScript” DSL | NS 조회 등 도메인 함수 제공→오픈·사용자 스크립팅 자유 |
| **Hubble** | 네트워크 관찰 | CNI 메타 연동 | gRPC Relay API | Service Map·CLI/UI 소비용 API 제공 |

### 5. Hands-On 사례 (Pixie)

- **데이터 보강**: DNS 트래픽에 파드명·네임스페이스 자동 추가
- **API 예시**:
    
    ```
    df = trace_dns()
    df = df.with_columns(
      pod_name=pod.get_pod(df.pod_id),
      hostname=df.remote_ip.ns_lookup()
    )
    df.publish()
    
    ```
    
- **확장 활용**: 새 로그파일 tail + BPF trace(OOM kill) 데이터를 PixieScript로 합쳐 “파드 OOM 발생·관련 syslog” 한눈에

### 6. 결론 & 권장

- **자동 계측**→쉬움, **Insight 제공**이 핵심
- 첫 도구는 **구조화 API + 보강 + 확장성** 갖춘 고수준 eBPF 프로젝트 추천
- 선택 시 위 세 축(보강·API·DSL/정책)을 검증해 “실제 운영 가시성” 확보하세요

> Q&A: 발표 뒤 남아 토론 가능! eBPF 도구 검토 중 궁금점 언제든 환영합니다.
>