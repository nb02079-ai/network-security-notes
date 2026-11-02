# 2026-09-03 · ASAv Failover(HA) · pfSense · 공격용 webterm 설정

> 원본: [original/0903_260907_091411_1.pdf](original/0903_260907_091411_1.pdf)

## 1. ASAv Failover (고가용성, HA)
방화벽 두 대(ASAv1/ASAv2)를 묶어 한 대가 죽으면 다른 대가 이어받게 한다.
토폴로지: ASAv1(.1) — gi0/2 — ASAv2(.2), Failover 링크로 gi0/2 사용.

### 1호기 (Primary)
```
conf t
 failover lan interface Failover gi0/2                    ! Failover 전용 인터페이스 지정
 failover interface ip Failover 172.16.0.1 255.255.255.0 standby 172.16.0.2
                                 └ active IP ┘            └ standby IP ┘
 failover key test1234                                     ! 두 장비 간 인증 키
 failover lan unit primary                                 ! 이 장비를 primary로
 failover                                                  ! failover 기능 활성화
 failover link Failover gi0/2                              ! 상태 동기화 링크
 show failover
```
- 데이터 인터페이스에도 active/standby IP를 쌍으로 준다:
```
conf t
 interface gi0/0
  ip address 192.168.250.2 255.255.255.0 standby 192.168.250.1
  nameif inside
 interface gi0/1
  ip address 1.1.1.2 255.255.255.0 standby 1.1.1.1
  nameif outside
```

### 2호기 (Secondary)
```
conf t
 failover lan interface Failover gi0/2
 failover interface ip Failover 172.16.0.2 255.255.255.0 standby 172.16.0.1
 failover key test1234
 failover lan unit secondary
 failover
 failover link Failover gi0/2
```
- 두 장비의 **key가 같아야** 하고, unit만 primary/secondary로 다르게.

## 2. pfSense — 오픈소스 방화벽/UTM
- **UTM(Unified Threat Management)**: 방화벽 + IDS/IPS 등 여러 보안 기능을 하나로 통합 관리.
- 인터페이스 2개 구성
  - **1번 어댑터**: 어댑터에 브리지(외부/WAN 쪽)
  - **2번 어댑터**: 내부 네트워크(LAN)
  - LAN DHCP 예: `192.168.16.14/24`

## 3. 공격 대상용 webterm 설정 (GNS3 내 리눅스 호스트)
```
ip address add 10.10.20.3/24 dev eth0     # 인터페이스에 IP 부여
ip link set eth0 up                        # 인터페이스 활성화
ip route add default via 10.10.20.1        # 기본 게이트웨이 설정
```
- `default via <게이트웨이>` = 목적지가 라우팅 테이블에 없을 때 보낼 기본 경로.

## 4. 정정·보충
- ASA Failover 종류: **Active/Standby**(1대만 처리) vs **Active/Active**(컨텍스트별 분담). 본 실습은 Active/Standby.
- Failover 링크로는 상태 동기화(세션 테이블 등)가 오가므로, 전용 인터페이스로 분리하는 것이 안정적.
- pfSense는 리눅스가 아니라 **FreeBSD 기반**. ClamAV·Snort/Suricata 패키지를 올려 UTM처럼 확장한다.
- `ip` 명령(iproute2)은 구형 `ifconfig`/`route`를 대체한다.

## 5. 복습 질문
1. ASA Failover에서 primary와 secondary가 공유해야 하는 값과, 달라야 하는 값은?
2. Active/Standby와 Active/Active의 차이는?
3. UTM이 단일 방화벽과 다른 점은?
4. `ip route add default via ...`는 어떤 트래픽에 적용되나?
