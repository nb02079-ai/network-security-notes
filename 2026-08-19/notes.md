# 2026-08-19 · VPN(GRE·IPsec) · FHRP(HSRP) · IPv6 · L3/L2 스위치

> 원본: [original/0819_260820_092958_1.pdf](original/0819_260820_092958_1.pdf)

## 1. VPN — Site-to-Site 구성

### GRE Tunnel
- **암호화 없이** 두 거점을 2계층처럼 이어주는 터널 방식 (캡슐화만, 기밀성 없음).
- 설정 순서: 터널 인터페이스 생성 → 터널 모드 지정 → 터널 출발지·목적지 지정 → 터널 IP용 라우팅 설정

```
! ① 터널 인터페이스 생성
Router(config)# interface tunnel 100
Router(config-if)# ip address 10.10.10.1 <subnet mask>    ! 터널 IP(양 끝 같은 대역)

! ② 터널 모드
Router(config-if)# tunnel mode gre ip                     ! 모드 종류: gre / ipv4 등

! ③ 출발지·목적지 (실제 물리 인터페이스·상대 공인 IP)
Router(config-if)# tunnel source se0/3/0                  ! 시작 인터페이스
Router(config-if)# tunnel destination 2.2.2.2            ! 목적지(상대) IP

! ④ 터널 너머 네트워크로 가는 경로
Router(config)# ip route 192.16.0.0 255.255.255.0 10.10.10.2
                         └ 목적지 네트워크 ┘ └ 마스크 ┘ └ 터널 건너편 IP ┘
```
- `tunnel mode gre` → **암호화·인증 기능이 없음** → 보안이 필요하면 IPsec.
- cmd의 `tracert <목적지 IP>` → 패킷 전송 경로(홉)를 표시.

### IPsec (IP Security)
GRE와 달리 **암호화·인증**을 제공. 설정 순서: Access-list → Crypto isakmp 정책 → transform-set → Crypto-map → 인터페이스에 map 적용

```
! ① 보호할 트래픽 정의 (ACL)
Router(config)# access-list 100 permit gre host <출발지> host <목적지>
   ! 또는  access-list 100 permit gre 192.168.160.0 0.0.0.255 host 2.2.2.2

! ② ISAKMP(IKE 1단계) 정책 — 키 교환·인증 방식
Router(config)# crypto isakmp policy 10
Router(config-isakmp)# authentication pre-share
Router(config-isakmp)# encryption 3des
Router(config-isakmp)# hash md5
Router(config)# crypto isakmp key 1234 address 2.2.2.2     ! 사전공유키(password) + 상대 IP

! ③ Transform-set — 실제 데이터 암호화/인증 방식 (IKE 2단계)
Router(config)# crypto ipsec transform-set test esp-3des esp-md5-hmac
                                               └ 암호화 ┘ └ 인증 ┘

! ④ Crypto-map — 위 요소들을 묶음
Router(config)# crypto map vpn-map 10 ipsec-isakmp
Router(config-crypto-map)# match address 100       ! 보호 대상 ACL
Router(config-crypto-map)# set peer 2.2.2.2         ! 상대 peer
Router(config-crypto-map)# set transform-set test

! ⑤ 인터페이스에 map 적용
Router(config)# interface se0/3/0
Router(config-if)# crypto map vpn-map
```
- 핵심 흐름: **ISAKMP(키 교환) → transform-set(데이터 보호) → crypto-map(묶어서 인터페이스 적용).**

## 2. FHRP (First Hop Redundancy Protocol, 게이트웨이 이중화)
- 여러 라우터를 묶어 **하나의 가상 게이트웨이**처럼 보이게 한다.
- 평소 **Active** 라우터가 트래픽 처리, 장애 시 **Standby**가 자동으로 넘겨받아(Failover) 통신 유지.
- 종류: **HSRP**(Cisco), **VRRP**(표준), **GLBP**(부하분산+이중화).

### HSRP 설정
```
Router(config)# interface gig0/0
Router(config-if)# standby 1 ip <가상 게이트웨이 IP>    ! 1 = 그룹 번호, PC들의 게이트웨이로 쓸 IP
Router(config-if)# standby 1 preempt                   ! 우선순위 높은 라우터가 복귀 시 Active 탈환
Router(config-if)# standby 1 priority 105              ! 우선순위(기본 100, 높을수록 Active)
```
- 그룹 번호를 다르게 하면 그룹마다 Active/Standby를 다르게 배치 → **부하 분산** 가능.

### 인터페이스 Tracking
- Active 라우터의 **외부(상류) 인터페이스 상태**를 추적해 장애 시 우선순위를 낮춰 Standby로 전환.
```
Router(config)# interface gig0/0
Router(config-if)# standby 1 track se0/3/0            ! 추적할 인터페이스
```
- se0/3/0에 장애가 생기면 우선순위가 **기본 10만큼 감소**해 Standby가 Active가 된다.

## 3. IPv6
- **128bit**, 16진수 표기.
- 주요 주소 범위
  - 글로벌 유니캐스트(공인): `2001::/16`, `2002::/16`, `3ffe::/16`
  - 링크 로컬(같은 링크 내부): `fe80::/10`
  - 사이트 로컬(구형, 현재 폐지됨)

```
Router(config)# interface se0/2/0
Router(config-if)# ipv6 address 2001:2:2:2::1/64
Router(config-if)# ipv6 ospf 1 area 0
Router(config)# ipv6 unicast-routing        ! IPv6 라우팅 기능 전역 활성화(필수)
Router(config)# ipv6 router ospf 1
Router(config-rtr)# router-id 11.11.11.11   ! IPv6 OSPF도 Router ID는 32bit(IPv4 형식)
```

## 4. L3 스위치 (라우팅 기능 활성화)
스위치 포트를 라우터 포트처럼 쓰려면:
```
Switch(config)# interface fa0/0
Switch(config-if)# no switchport           ! 라우티드 포트로 전환
Switch(config-if)# ip address <ip> <mask>
Switch(config)# ip routing                 ! 라우팅 기능 전역 활성화
```

## 5. L2 스위치 — 관리용 접속 설정
L2 스위치는 포트에 IP를 못 주는 대신 **관리용 가상 인터페이스(VLAN)** 에 IP를 준다.
```
Switch(config)# interface vlan 1
Switch(config-if)# ip address <ip> <subnet>
Switch(config-if)# no shutdown
Switch(config)# ip default-gateway <gateway IP>     ! 스위치가 외부로 나갈 게이트웨이
Switch(config)# line vty 0 4                         ! 원격 접속 설정
Switch(config-line)# login
Switch(config-line)# password 1234
Switch(config)# enable password 1234
```
- 다른 end device에서 `telnet <스위치 IP>`로 원격 접속 확인.

## 6. 정정·보충
- 원본 "First Map Redundancy Protocol" → 정확한 명칭은 **First Hop** Redundancy Protocol.
- 3des·md5는 학습·실습용 조합이며, 실무에서는 취약하므로 **AES + SHA** 계열을 쓴다.
- IPsec 2단계(데이터 보호) 방식 ESP는 암호화+인증, AH는 인증만 제공.

## 7. 복습 질문
1. GRE와 IPsec의 결정적 차이는 무엇인가?
2. HSRP에서 preempt 옵션이 없으면 어떤 일이 생기나?
3. IPv6에서 `ipv6 unicast-routing`을 빠뜨리면 무슨 문제가 생기나?
4. L2 스위치에 IP를 직접 못 주는 이유와, 그 대안은?
