# Network Traffic Quality of Service (QoS) Classes for Containers & Pods Via EBPF/XDP

다중 선택: Linux Security Summit 2023 NA

**발표 요약: “컨테이너·파드용 네트워크 QoS 클래스 구현 (eBPF/XDP)”**

1. **연사 & 배경**
    
    – Vinay & Phu, Futurewei Cloud Lab
    
    – 관심 분야: Kubernetes, CNI, eBPF/XDP 기반 고성능 네트워킹
    
2. **QoS 개요 & 필요성**
    
    – 실시간·스트리밍·데이터 전송 등 애플리케이션별 네트워크 요구(지연·손실·대역폭) 상이
    
    – 컨테이너 환경에서도 ‘누가 더 중요한 트래픽인가?’ 분류·우선순위 지정이 필수
    
3. **eBPF/XDP 개념**
    
    – eBPF: 커널 내 안전한 사용자 정의 프로그램 실행 플랫폼
    
    – XDP: NIC 수신 경로 최상단에서 실행되는 eBPF 후크로, 스택 할당 전 초경량 패킷 처리
    
4. **MisaR 프로젝트**
    
    – CNI 플러그인: Geneve 오버레이로 멀티테넌시 네트워킹 제공
    
    – Node 에이전트(Transit Daemon) ⇄ eBPF 맵
    
    – XDP 프로그램: veth/호스트 NIC 간 ‘fast path’(redirect) vs ‘slow path’(pass→네트워크 스택) 분기
    
5. **QoS 설계**
    
    – 세 가지 클래스: **Premium ▶ Expedited ▶ Best-Effort**
    
    – 1) **분류(Classification)**: Pod annotations→Transit Daemon→eBPF 맵에 (srcIP, port)→class 등록
    
    – 2) **조치(Action)**:
    
    - Premium→XDP redirect→즉시 egress
    - Expedited→XDP redirect(2nd tier)
    - Best-Effort→XDP pass→네트워크 스택(priority queueing + eBPF 기반 동적 Token-bucket rate limiting)
        
        – 부가: DSCP 필드 설정으로 인프라 전구간 ‘end-to-end QoS’
        
6. **데모 & 결과**
    
    – 3개의 Sender VM(각각 Premium/Expedited/Best-Effort), 3개의 Receiver VM
    
    – 동시 전송 시 Premium≈900 Mbps, Expedited≈800 Mbps, Best-Effort≈400 Mbps 보장
    
    – Best-Effort 먼저 시작→Premium 투입 시 Best-Effort ≈10–20 %로 자동 제한, Premium 전용 대역 확보
    
7. **향후 과제**
    
    – XDP TX 후크(offload) 지원으로 더 낮은 지연·CPU 오버헤드
    
    – K8s API(예: PodQoS) 공식화 제안
    
    – CNI-Genie 통합, 서비스 메시 호환성 검토
    
    – Control-plane 성능 측정, 동적 스케일링·해시 분산, NUMA-aware 설계 등
    

> 핵심: eBPF/XDP 기반 CNI 플러그인으로 클래스별 트래픽 분류(Classification) → 우선순위 큐·동적 Rate-limiting(Action) 구현, 쿠버네티스 파드 단위의 QoS 보장.
>