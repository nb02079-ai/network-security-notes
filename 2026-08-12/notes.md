# 2026-08-12 · 정보보안 개론 · OSI/TCP-IP · IP 주소 · Packet Tracer 기초

> 원본: [original/0812_260812_174551_1.pdf](original/0812_260812_174551_1.pdf)

## 1. 보안 분야 진로 메모
- 주요 직무: 보안 엔지니어 / 보안 관제 / 보안 컨설팅 / 분석(포렌식·악성코드) 엔지니어 / 모의해킹
- 자격 경로: 정보처리·정보보안 기사 + 실무 경력 → 기술사 응시
- 정보시스템 감리사: 기술사급 자격 + 감리 연수 이수
- 실무형 자격: **OSCP** (모의해킹 실기 중심)

## 2. 정보보안 기초
| 구분 | 내용 |
|---|---|
| 관리적 보안 | 정책·규정·교육 등 문서화된 통제 (인적 보안 포함) |
| 기술적 보안 | 네트워크 보안 / 시스템 보안 / 웹 보안 — 실습의 중심 |
| 물리적 보안 | 출입 통제, 장비·시설 보호 |

- 대부분의 보안 사고는 **사람의 실수**에서 시작된다.
- 클라우드 서비스: 인터넷만 되면 어디서든 원하는 작업(자원 사용)을 할 수 있는 것.

### 보안의 3요소 (CIA)
- **기밀성(Confidentiality)**: 암호화 등으로 인가되지 않은 사람이 볼 수 없게 한다.
- **무결성(Integrity)**: 변조·위조 없이 원본 그대로 유지되어야 한다.
- **가용성(Availability)**: 필요할 때 언제든 서비스를 이용할 수 있어야 한다.

## 3. OSI 7계층 vs TCP/IP 4계층
| OSI | 역할 | 예시 | TCP/IP |
|---|---|---|---|
| 7 응용 | 사용자 인터페이스 제공 | HTTP, DNS, FTP | Application |
| 6 표현 | 암호화, 압축, 인코딩 | TLS, JPEG | Application |
| 5 세션 | 통신 연결 관리 | NetBIOS, RPC | Application |
| 4 전송 | 종단 간 전달 | TCP, UDP | Transport |
| 3 네트워크 | 경로 선택(라우팅) | IP, ICMP / 라우터 | Internet |
| 2 데이터 링크 | 같은 네트워크 내 전송 | Ethernet, ARP / 스위치 | Network Access |
| 1 물리 | 비트 전송 | 케이블, 허브 | Network Access |

- 송신 측은 위→아래로 **캡슐화**, 수신 측은 아래→위로 **역캡슐화**.
- 외우기: Physical – Data link – Network – Transport – Session – Presentation – Application

### TCP vs UDP
| | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
|---|---|---|
| 연결 | 3-way handshake (SYN → SYN/ACK → ACK) | 연결 설정 없음 |
| 신뢰성 | 확인 응답·재전송·순서 보장 | 확인 절차 없음 |
| 용도 | 웹, 메일, 파일 전송 | 스트리밍, DNS 질의, VoIP |

## 4. 주소 체계
- **MAC 주소**: 하드웨어(NIC) 주소, 48bit, 2계층
- **IP 주소**: 논리(소프트웨어) 주소, 3계층
- **ARP (Address Resolution Protocol)**: IP 주소 → MAC 주소를 알아내는 프로토콜
- **IPv4**: 32bit, 10진수 4옥텟 표기 (예: 192.168.16.1)
- **IPv6**: 128bit, 16진수 8그룹 표기 (예: 2001:db8::1)
- 키워드: 서브넷 마스크 / 서브네팅 / 슈퍼네팅, 주소 변환(NAT·NAPT), 사설망과 보안, 가상 사설망(VPN)

## 5. Cisco Packet Tracer 기초
- 토폴로지: **물리적(Physical)** / **논리적(Logical)**
- 케이블
  - Copper **Straight-Through**: 서로 다른 계층 장비 연결 (PC ↔ 스위치, 스위치 ↔ 라우터)
  - Copper **Cross-Over**: 같은 종류 장비 연결 (PC ↔ PC, 스위치 ↔ 스위치, 라우터 ↔ PC)
- LAN(Local Area Network) / WAN(Wide Area Network)
- 스위치는 프레임의 출발지 MAC을 보고 **MAC 주소 테이블**을 학습한다. 같은 네트워크 대역에 IP를 주면 PC1 ↔ PC2 통신 가능.
- 잘못 입력한 명령이 도메인 조회로 멈추면 `Ctrl + Shift + 6`으로 중단.
- 패킷은 라우터(hop)를 하나 지날 때마다 **TTL이 1씩 감소**하고, 0이 되면 폐기(drop)된다 → 라우팅 루프 방지.

## 6. 라우터 기본 설정 (Cisco IOS)
```
Router> enable                     ! 관리자 모드 (en)
Router# configure terminal         ! 전역 설정 모드 (conf t)
Router(config)# line console 0     ! 콘솔 0번 설정
Router(config-line)# password cisco
Router(config-line)# login
Router(config)# enable password 1234   ! 평문 저장
Router(config)# enable secret 1234     ! 암호화 저장, password보다 우선 적용
Router(config)# line vty 0 4           ! 원격 접속(Telnet/SSH) 5세션
Router(config)# service password-encryption   ! 평문 비밀번호를 type 7로 암호화
```
- `?` 명령어 도움말, `Tab` 자동완성
- `show running-config`: 현재 동작 중인 설정(RAM)
- `show startup-config`: 재부팅 시 적용될 설정(NVRAM)
- 저장: `copy running-config startup-config` 또는 `write`

### 인터페이스(게이트웨이) 설정
```
Router(config)# interface gig0/0
Router(config-if)# ip address 192.168.16.1 255.255.255.0
Router(config-if)# no shutdown       ! 인터페이스 활성화
```
- IP 할당 방식: **DHCP**(Dynamic Host Configuration Protocol, 자동) / **Static**(수동)
- 192.168.16.1/24 대역: .0은 네트워크 주소, .255는 브로드캐스트 → 호스트는 .1~.254
- `/24` 프리픽스 표기 = 서브넷 마스크를 2진수로 쓸 때 1의 개수 (255.255.255.0)

## 7. 정정·보충
- 원본의 "enable password / enable secret 절대 동일" → 둘을 같이 쓰면 **secret이 우선**이고, IOS는 두 값이 같으면 경고를 준다. 실무에서는 secret만 사용.
- `service password-encryption`의 type 7 암호화는 쉽게 복호화되므로 보안 수단이 아니라 어깨너머 노출 방지용이다.
- 참고 자료(원본 마지막 장): 유튜브 「에듀웨어 – 네트워크 여행」

## 8. 복습 질문
1. 무결성과 기밀성의 차이를 예시로 설명할 수 있나?
2. 스위치와 라우터는 OSI 몇 계층 장비인가?
3. running-config와 startup-config의 차이는?
4. 192.168.16.0/24에서 PC에 줄 수 없는 주소 두 개는?
