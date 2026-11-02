# 2026-09-07 · pfSense 라우팅·방화벽 규칙 (GNS3 연동)

> 원본: [original/Notes_260907_120210_1.pdf](original/Notes_260907_120210_1.pdf)

09-03의 pfSense 도입에 이어, **GNS3에 pfSense를 연동해 라우팅과 방화벽 규칙**을 설정한다.

## 1. pfSense = 라우팅 + 게이트웨이 + 방화벽
- **UTM(Unified Threat Management)**: 통합 위협 관리 = 방화벽 + IDS/IPS 등을 한 장비에서.
- 토폴로지: `pfSense`가 상단, 아래에 R1·R2 연결 (pfSense가 경계에서 라우팅/필터링).

## 2. 정적 라우팅 (System → Routing)
pfSense 웹 UI의 **System > Routing > Static Routes**에서 edit으로 경로 설정.

| Destination network | Gateway |
|---|---|
| 192.168.100.0 | 1.1.1.1 |
| 192.168.200.0 | 2.2.2.1 |

- 목적지 네트워크마다 다음 홉(게이트웨이)을 지정 → CLI의 `ip route`와 같은 개념을 GUI로.

## 3. 방화벽 규칙 (Firewall → Rules)
**Firewall > Rules > edit**에서 규칙 설정.
- 기본 정책: **WAN에서 들어오는 모든 통신은 기본 차단** (상단 규칙부터 순서대로 평가).

### 차단 동작 2가지
| 동작 | 결과 |
|---|---|
| **Block** | 패킷을 조용히 버림 (응답 없음 → 공격자가 포트 상태를 알기 어려움) |
| **Reject** | 거절 응답을 보냄 (TCP RST / ICMP unreachable 등, 로그에 남음) |

> 일반적으로 외부(WAN) 대상은 **Block**(스텔스), 내부(LAN)에서 사용자 편의가 필요하면 **Reject**로 즉시 거절을 알리는 식으로 구분한다.

## 4. 정정·보충
- pfSense 규칙은 **인터페이스별(WAN/LAN)로 방향이 "들어오는 기준(inbound)"** 으로 적용된다 — Cisco ACL의 in/out과 사고방식이 다르다.
- 규칙은 위에서 아래로 평가하되, 먼저 일치하는 규칙에서 처리(first-match). 순서가 중요.
- Block은 로그가 남지 않게도 설정 가능하지만, 보안 분석을 위해 로깅을 켜두는 편이 좋다.

## 5. 복습 질문
1. 방화벽에서 Block과 Reject의 차이, 각각 언제 쓰나?
2. pfSense 규칙의 평가 순서 원칙은?
3. UTM과 단순 패킷 필터 방화벽의 차이는?
