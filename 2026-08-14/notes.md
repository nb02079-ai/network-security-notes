# 2026-08-14 · EIGRP · 재분배 · VLAN · Router-on-a-stick · DHCP

> 원본: [original/0814_260814_223213.pdf](original/0814_260814_223213.pdf)

## 1. EIGRP (Enhanced Interior Gateway Routing Protocol)
```
Router(config)# router eigrp 10                      ! 10 = AS(Autonomous System) 번호
Router(config-router)# network 192.168.100.0 0.0.0.31  ! 네트워크 + 와일드카드 마스크
Router(config-router)# no auto-summary
```
- **와일드카드 마스크** = 서브넷 마스크의 0↔1 반전 (255.255.255.224 → 0.0.0.31)
- 경로 선택: **대역폭(bandwidth) + 지연(delay)** 으로 메트릭 계산 (신뢰도·부하는 기본 미사용)
- 최대 홉: 기본 100, 최대 255까지 설정 가능
- AD: 내부 90 / 외부(재분배) 170
- **AS 번호가 다르면 라우팅 정보를 공유하지 않는다** → 재분배 필요

## 2. 재분배(Redistribution)
- **ASBR**: 서로 다른 AS(라우팅 도메인)의 경계에 있는 라우터
- Packet Tracer에서는 경계 라우터 한 곳에서 양방향 재분배하면 된다

```
! 경계 라우터에 eigrp 10, 20이 모두 설정되어 있다고 가정
Router(config)# router eigrp 20
Router(config-router)# redistribute eigrp 10
Router(config-router)# exit
Router(config)# router eigrp 10
Router(config-router)# redistribute eigrp 20
Router(config-router)# network <경계 대역 IP> <와일드카드>
```
- 서로 다른 라우팅 프로토콜에 **같은 대역을 중복 광고하지 않도록** network 문을 확인한다.

## 3. VLAN (Virtual LAN)
- 하나의 물리 스위치를 논리적으로 분리. 분리된 VLAN끼리 통신하려면 **VLAN마다 게이트웨이(라우터)** 가 필요하다.

| 할당 방식 | 특징 |
|---|---|
| MAC 기반 | 장비가 바뀔 때마다 재설정 |
| IP 기반 | IP가 바뀌거나 추가될 때마다 재설정 |
| **Port 기반** | 연결된 포트에 따라 배정 — **가장 많이 사용** |
| Policy 기반 | 정책 조건에 따라 배정 |

- 스위치 ↔ 스위치, 스위치 ↔ 라우터: **trunk** / 스위치 ↔ 단말: **access**
- 해당 스위치 아래에 그 VLAN 단말이 없어도, 전체 망에 존재하는 VLAN은 모든 스위치에 생성해 두는 게 안전하다(트렁크로 전달되도록)
- 같은 VLAN끼리만 통신, 다른 VLAN은 라우팅 없이 통신 불가

```
Switch(config)# vlan 10
Switch(config-vlan)# name network
Switch(config-vlan)# exit
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config)# interface fa0/24
Switch(config-if)# switchport mode trunk
```

### Router-on-a-stick (VLAN별 게이트웨이)
```
Router(config)# interface gig0/0.10              ! 서브인터페이스 번호 = 보통 VLAN 번호
Router(config-subif)# encapsulation dot1Q 10     ! 802.1Q 태그 VLAN 10
Router(config-subif)# ip address 192.168.200.50 255.255.255.x
Router(config)# interface gig0/0
Router(config-if)# no shutdown                   ! 물리 인터페이스를 켜야 서브인터페이스도 동작
```

## 4. DHCP
동작 순서(DORA): **Discover**(브로드캐스트) → **Offer**(IP·마스크·게이트웨이 제안) → **Request** → **Ack**(확인 응답)

```
Router(config)# ip dhcp excluded-address 192.168.100.1 192.168.100.10   ! 풀에서 제외
Router(config)# ip dhcp pool dhcptest
Router(dhcp-config)# network 192.168.100.0 255.255.255.224
Router(dhcp-config)# default-router 192.168.100.1
Router(dhcp-config)# dns-server 8.8.8.8
```
- DHCP 서버는 MAC 주소를 기억해 **가능하면 이전에 받던 IP를 다시 준다**
- 서버와 클라이언트가 다른 네트워크에 있을 때: **DHCP Relay** — 클라이언트 쪽 게이트웨이 인터페이스에 `ip helper-address <DHCP 서버 IP>`

### /26 예시: 네트워크 주소와 게이트웨이(첫 주소)
| 범위 | 네트워크 | 게이트웨이 |
|---|---|---|
| 0~63 | .0 | .1 |
| 64~127 | .64 | .65 |
| 128~191 | .128 | .129 |
| 192~255 | .192 | .193 |

- 연결된 인터페이스 IP 등록 필수, EIGRP에서는 연결된 네트워크 주소를 광고해야 한다.

## 5. 정정·보충
- 원본 표의 64~127 게이트웨이 "66" → 관례상 첫 사용 가능 주소인 **.65**.
- EIGRP 최대 홉 255는 설정 가능 최댓값이고 기본값은 100.
- `ip dhcp relay`라는 명령보다 실제 설정은 `ip helper-address`.

## 6. 복습 질문
1. 192.168.100.0/27의 와일드카드 마스크는?
2. 트렁크 포트와 액세스 포트의 차이는?
3. DHCP DORA 과정 중 브로드캐스트로 나가는 메시지는?
