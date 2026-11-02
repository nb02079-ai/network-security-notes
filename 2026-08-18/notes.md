# 2026-08-18 · OSPF · 프로토콜 간 재분배 · ACL · NAT/PAT · VPN 개요

> 원본: [original/0818_260820_141936_1.pdf](original/0818_260820_141936_1.pdf)

## 1. OSPF (Open Shortest Path First)
- EIGRP가 AS 번호로 묶는 것과 달리 OSPF는 **Area 번호**로 영역을 나눈다.
- **Area 0 = 백본(backbone) 영역.**
- Area 번호가 달라도 같은 OSPF면 **재분배 불필요** (영역 경계 라우터 ABR이 이어줌).
- 구성 방식
  - **Single Area**: 영역을 하나만 두는 경우 → 그 영역을 Area 0으로 설정.
  - **Multi Area**: 모든 영역이 물리적·논리적으로 **백본(Area 0)에 연결**되어야 한다.
- **Virtual Link**: 백본과 직접 붙지 못하는 Area를 가상 링크로 Area 0에 연결.

```
Router(config)# router ospf 1                               ! 1 = 프로세스 번호(로컬 의미)
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
                               └ 네트워크 IP ┘ └ 와일드카드 ┘ └ Area ┘
Router(config-router)# do show ip protocols
```
- 두 라우터를 잇는 경계 대역은 **양쪽이 같은 Area 번호**로 광고해야 한다 (예: Router2–Router3 링크가 Area 0이면 둘 다 area 0).

### Router ID 선정 순서
1. 관리자가 `router-id`로 직접 지정
2. **Loopback** 인터페이스에 설정된 IP 중 가장 높은 값
3. 활성화된 물리 인터페이스 IP 중 가장 높은 값

```
Router(config)# interface loopback 0
Router(config-if)# ip address <ip> <subnet mask>
Router(config)# router ospf 1
Router(config-router)# router-id 11.11.11.11
Router# clear ip ospf process        ! 바꾼 Router ID 적용하려면 프로세스 재시작
```
- Loopback은 물리 케이블·포트와 무관한 **논리 가상 인터페이스** → 항상 up 상태라 Router ID로 안정적.
- **확장형 ping**: `Router# ping` 만 입력하면 출발지·개수 등을 대화형으로 지정 가능 (loopback을 출발지로 도달성 테스트).

## 2. 서로 다른 라우팅 프로토콜 간 재분배
- 경계 대역은 양쪽 프로토콜 모두에 광고해야 한다.

```
! eigrp 100이 설정된 경계 라우터에서 ospf 1을 끌어오기
Router(config)# router eigrp 100
Router(config-router)# redistribute ospf 1 metric 1544 2000 255 1 1500
                                            └ BW(Kbps) Delay 신뢰도 Load MTU ┘
                                              ! EIGRP는 메트릭 5요소를 지정해야 함
Router(config)# router ospf 1
Router(config-router)# redistribute eigrp 100 subnets
                                              ! OSPF로 끌어올 때는 subnets 키워드
```
- 재분배 방향 이해: `ospf1 → (eigrp100 광고)`, `eigrp100 → (ospf1 광고)` 처럼 **양방향**으로 서로의 경로를 교환.

## 3. ACL (Access Control List, 접근제어목록)
Network 보안의 시작. 조건에 따라 패킷을 **허용(permit)/차단(deny)**.

| 종류 | 분류 기준 |
|---|---|
| **Standard** | **출발지 네트워크(IP)** 하나로만 판별 — 번호 1~99 |
| **Extended** | 출발지·목적지·프로토콜·포트 등 **세부 조건**으로 판별 — 번호 100~199 |

### 핵심 원칙
- Standard는 **목적지에 가장 가까운 라우터**에, Extended는 **출발지에 가장 가까운 라우터**에 둔다 (불필요한 트래픽을 일찍/늦게 거르는 설계).
- 인터페이스에 **적용(access-group)해야** 비로소 동작한다.
- 정책은 **위에서부터 순서대로** 평가 → 조건에 맞으면 거기서 멈추고 적용.
- ACL 맨 끝에는 보이지 않는 **`deny any`(암묵적 전체 차단)** 가 있다 → 허용할 대상은 반드시 `permit`을 명시해야 한다.

### Standard ACL — 번호 방식
```
Router(config)# access-list 10 deny 192.16.100.3 0.0.0.0    ! 번호 deny 출발지IP 와일드카드
Router(config)# access-list 10 permit any
Router(config)# interface gig0/0
Router(config-if)# ip access-group 10 out       ! 방향 지정
```
- `in` = 인터페이스로 **들어올 때** 검사 / `out` = 인터페이스에서 **나갈 때** 검사.
- 확인: `do show access-lists`

### Standard ACL — 이름(named) 방식
```
Router(config)# ip access-list standard test
Router(config-std-nacl)# deny 192.16.100.3 0.0.0.0
Router(config-std-nacl)# permit any
Router(config)# interface gig0/0
Router(config-if)# ip access-group test out      ! 번호 대신 이름으로 적용
```

### Extended ACL
```
Router(config)# access-list 100 deny icmp host <출발지IP> host <목적지IP>   ! 특정 호스트 간 ping 차단
Router(config)# access-list 100 permit ip any any
Router(config)# interface gig0/0
Router(config-if)# ip access-group 100 out
```

## 4. NAT / PAT
- **NAT (Network Address Translation)**: 네트워크 주소 변경 — 기본적으로 **1:1** 변환.
- **PAT (Port Address Translation)**: 포트 번호까지 함께 변경 → **다:1** 변환 (여러 사설 IP가 공인 IP 하나를 포트로 구분해 공유). 가정 공유기가 PC·노트북·폰을 한 공인 IP로 내보내는 방식.
- **사설 IP는 그대로 인터넷에 못 나가므로 공인 IP로 변환**해야 외부와 통신 가능.
- inside = 내부(사설) 쪽, outside = 외부(공인) 쪽.

```
Router(config)# ip nat pool dyn 1.1.1.1 1.1.1.254 netmask 255.255.255.0
                            └ 풀 이름 ┘└ 공인 시작 ┘└ 공인 끝 ┘
Router(config)# access-list 10 permit <사설네트워크> <와일드카드>    ! 변환 대상 지정
Router(config)# ip nat inside source list 10 pool dyn
                            ! list 10에 해당하는 내부 출발지를 dyn 풀의 공인 IP로 변환
Router(config)# interface gig0/0
Router(config-if)# ip nat inside            ! 내부 인터페이스 지정
Router(config)# interface se0/3/0
Router(config-if)# ip nat outside           ! 외부 인터페이스 지정
```
- 확인: `show ip nat statistics`, `show ip nat translations`
- **Static NAT** (고정 1:1):
```
Router(config)# ip nat inside source static <사설IP> <공인IP>
```

## 5. VPN (Virtual Private Network) 개요
공용망 위에 가상의 사설 통신망을 만드는 기술. 두 가지 구성:
- **Site-to-Site**: 두 거점(지사–본사) 네트워크를 통째로 연결
- **Remote-Access**: 개별 사용자가 원격에서 사내망에 접속

(상세 설정 — GRE 터널, IPsec — 은 08-19 노트에서 이어짐)

## 6. 정정·보충
- OSPF 프로세스 번호(`router ospf 1`의 1)는 **로컬 의미**로, 이웃 라우터와 같을 필요가 없다. 반면 Area 번호는 맞춰야 한다.
- Extended ACL 번호 범위는 100~199(+2000~2699 확장), Standard는 1~99(+1300~1999).
- PAT는 Cisco에서 `ip nat inside source list ... overload` 처럼 **overload** 키워드로 설정하는 것이 정석.

## 7. 복습 질문
1. Standard ACL과 Extended ACL을 각각 어느 위치에 두는 것이 좋은가, 그 이유는?
2. ACL 맨 끝의 암묵적 규칙은 무엇인가?
3. NAT와 PAT의 변환 비율(1:1 / 다:1) 차이는?
4. OSPF에서 Virtual Link가 필요한 상황은?
