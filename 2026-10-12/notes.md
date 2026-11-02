# 2026-10-12 · bWAPP(Bee-Box) ② — XSS 변형 · 구성/접근제어/DoS 취약점

> 원본: [original/Notes_261012_093412_1.pdf](original/Notes_261012_093412_1.pdf) (오전, XSS) · [original/Notes_261012_140912_1.pdf](original/Notes_261012_140912_1.pdf) (오후, 기타)
>
> ⚠️ bWAPP/Bee-Box 실습. 취약점 유형 카탈로그 + 방어. 실제 시스템 대상 사용 금지.

## 1. XSS 변형 (주입 지점별)
입력이 어디로 들어가느냐에 따라 XSS 벡터가 달라진다.

### Reflected XSS 주입 지점
| 지점 | 비고 |
|---|---|
| Email / HREF / Login-Form | 폼 필드·링크 속성 |
| **phpMyAdmin** | CVE-2010-4480 (알려진 취약점) |
| PHP_SELF | `$_SERVER['PHP_SELF']` 반사 |
| Referer / User-Agent | **HTTP 헤더** 기반 — 프록시로 헤더 조작해 삽입 |

### Stored XSS 주입 지점
- Change-Secret / Cookies / **SQLite Manager** / User-Agent 등 — 저장 후 조회 시 실행.

- 방어: 모든 입력(헤더 포함)을 신뢰하지 말 것 → **출력 인코딩 + CSP**. 헤더 기반(Referer/User-Agent)도 그대로 출력 금지.

## 2. 접근제어 / 참조 취약점
| 유형 | 설명 | 방어 |
|---|---|---|
| **Insecure DOR (IDOR)** | 식별자(order/secret)를 바꿔 남의 자원 접근. 변형: change/reset secret, order tickets — "인증은 받았으나 **권한(인가)** 검증 누락" | 객체 단위 인가 검증 |
| **Arbitrary File Access** | 임의 파일 접근 (metasploit-framework 활용) | 경로 검증, 권한 분리 |
| **Cross-Domain Policy File (Flash)** | `crossdomain.xml` 과대 허용 | 도메인 최소 허용 |
| **CORS (AJAX)** | Cross-Origin 리소스 공유 오설정 | Origin 화이트리스트 |
| **Cross-Site Tracing (XST)** | TRACE 메서드로 쿠키 탈취 | TRACE 비활성 |

## 3. DoS (Denial of Service) 변형
| 벡터 | 설명 |
|---|---|
| Large Chunk Size | 과대 청크로 자원 소모 |
| Slow HTTP DoS | 느린 요청으로 연결 고갈(Slowloris류) |
| SSL-Exhaustion | SSL 핸드셰이크 반복으로 CPU 고갈 |
| XML-Bomb | 엔티티 확장(Billion Laughs)으로 메모리 고갈 |
- 방어: 요청 크기·타임아웃 제한, 연결 수 제한, XML 엔티티 확장 비활성.

## 4. 구성(Configuration) 취약점
| 유형 | 점검 도구/방법 |
|---|---|
| Insecure FTP Configuration | msfconsole 사전 대입 |
| Insecure SNMP Configuration | `snmpwalk`로 정보 수집 |
| Insecure WebDAV Configuration | `cadaver` (davtest) — 코드 업로드 |
| Local Privilege Escalation (udev 등) | 로컬 권한 상승 |
| **MITM attack (HTTP/SMTP)** | **Wireshark**로 평문 스니핑 |
| Old/Backup & Unreferenced Files | 백업·미참조 파일 노출 |
| Robots File | robots.txt로 경로 노출 |
- 방어: 평문 프로토콜 제거(암호화), 기본 자격증명 변경, 불필요 서비스·파일 제거, SNMP community 변경.

## 5. 복습 질문
1. Referer·User-Agent 기반 XSS가 성립하는 이유는?
2. IDOR의 근본 원인(인증 vs 인가)은?
3. XML-Bomb이 DoS를 일으키는 원리는?
4. MITM으로 평문이 노출되지 않게 하려면?
