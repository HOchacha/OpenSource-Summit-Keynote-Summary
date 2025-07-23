# Network Intrusion Detection 101 with OpenWrt and OpenCanary - Tamas Lengyel, Intel

다중 선택: OSS 2024 NA

## 1. 발표자 및 주제

- **발표자**: 홈 네트워크 보안 애호가
- **주제**: OpenWrt + OpenCanary 기반 “네트워크 침입 탐지 101”

## 2. 동기 & 위협 모델

- **IoT 기기 증가**: 대부분 제조사·펌웨어를 신뢰할 수 없음
- **주요 공격 흐름**
    1. 장치 취약점 악용 → 네트워크 진입
    2. 내부 스캐닝(포트·서비스 검색)
    3. 사이드 이동 및 데이터 접근
- **목표**:
    - 가짜(허니) 서비스 배치 → 공격자 스스로 드러나게 함
    - 로그/알림 + 자동 대응(와이파이 차단)

## 3. 핵심 도구

1. **OpenWrt** (라우터 OS)
    - 수백 기종 지원, 패키지 시스템(`opkg`)
    - Docker, VLAN, 방화벽, RPC API(rpcd) 확장 가능
2. **OpenCanary** (오픈소스 허니팟)
    - FTP, SSH, HTTP, MySQL, Git 등 다수 서비스 에뮬레이션
    - 웹훅·이메일·Slack 알림 지원

## 4. 홈 라우터를 허니팟 호스트로 전환

1. **OpenWrt에 Docker 설치**
    
    ```bash
    opkg update
    opkg install dockerd docker-compose luci-app-docker
    service dockerd start
    docker pull thinkst/opencanary
    
    ```
    
2. **OpenCanary 설정** (`/etc/opencanary.conf`)
    
    ```json
    { "ssh.enabled": true,
      "ftp.enabled": true,
      "smtp.enabled": true,
      "webhook.enabled": true,
      "webhook.url":   "http://<router-ip>/ubus/service/opencanary",
      "webhook.magic":"<your-secret-token>"
    }
    
    ```
    
3. **허니팟 컨테이너 실행**
    - Web UI 혹은 CLI로 `/mnt/sda1/opencanary:/root/.opencanary` 마운트
    - `docker run --restart always … thinkst/opencanary`

## 5. 네트워크에 가짜 서비스 배치

1. **포트 포워딩** (LAN → Docker)
    - 방화벽 규칙: 외부 LAN 포트 21 → 컨테이너 내부 포트 21
2. **vLAN 기반 가짜 IP/MAC**
    - `kmod-dmac-vlan` 패키지 설치 → Mac-VLAN 인터페이스 생성
    - 인터페이스에 별도 IP 할당 + 방화벽 LAN 존 추가
    - 각 허니 서비스마다 고유 IP·MAC 사용 → 스캔 시 진짜 분간 불가

## 6. 자동화된 탐지 & 차단

1. **OpenCanary 웹훅 서비스** (`/usr/libexec/rpcd/opencanary`)
    
    ```bash
    #!/bin/sh
    magic="YourSecretToken"
    if [ "$1" != "list" ]; then
      [ "$2" = "$magic" ] && \
      ip=$(echo "$3" | jq -r .source_ip) || exit 1
      mac=$(ubus call network.device status "{\"name\":\"br-lan\"}" \
            | jq -r --arg ip "$ip" '.devices[] | select(.ipaddr==$ip).macaddr')
      uci add wireless.maclist=deny
      uci set wireless.@deny[-1].mac="$mac"
      uci commit wireless
      wifi reload
    fi
    
    ```
    
2. **알림 & 차단 흐름**
    - 공격자 → 가짜 서비스 → OpenCanary 웹훅 → OpenWrt API 호출
    - 해당 MAC 자동 차단(wifi maclist deny) + 실시간 이메일/Slack 알림

## 7. 확장 아이디어

- **추가 허니 서비스**: VNC, MySQL, Git 등
- **허니 토큰 연동**: 가짜 자격증명 발행→접속 시 탐지
- **포트 스캔 탐지**: OpenCanary 내장 기능 활용
- **대시보드 구축**: 실시간 접속 로그·차단 현황 시각화
- **네트워크 전체 퍼짐**: 내부 IP 전부에 허니 서비스 자동 응답

## 8. 요약

- **간단 구축**: OpenWrt + Docker + OpenCanary → 홈 라우터에서 즉시 허니팟 운영
- **수동 점검 부담 無**: 이메일 알림 + 자동 와이파이 차단으로 실시간 방어
- **공격자에 청신호 제공**: 스캔·접속 시 허니 서비스로 유도, 내부 침투 시도 공개
- **적은 유지보수**로 강력한 수동 탐지·차단 체계 완성