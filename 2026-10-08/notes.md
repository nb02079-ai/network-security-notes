# 2026-10-08 · bWAPP(Bee-Box) ① — Injection 계열

> 원본: [original/Notes_261008_093711.pdf](original/Notes_261008_093711.pdf)
>
> ⚠️ **bWAPP / Bee-Box**(의도적으로 취약한 웹앱 모음) 실습. 취약점 유형을 정리하고 방어를 함께 적는다. 실제 시스템 대상 사용 금지.

## 1. HTML Injection (GET / POST / Stored)
악성 HTML을 입력해 페이지에 삽입되는 취약점.
| 난이도 | 우회 |
|---|---|
| medium | `<`,`>` 기호 → HTML 엔티티로 변환됨. **URL 인코딩**해서 삽입하면 실행됨 |
| high | `htmlspecialchars()` 함수로 인해 HTML 엔티티로 변환 → 삽입 실패 |
- 변형: HTML injection **GET/POST/Current URL/Stored(Blog)** — 입력 경로만 다름.
- 방어: 출력 시 `htmlspecialchars()`/엔티티 인코딩, 입력 검증.

## 2. 그 외 Injection 유형 (카탈로그)
| 유형 | 설명 | 방어 |
|---|---|---|
| **iFrame Injection** | iframe 태그 삽입으로 외부 페이지 로드/클릭재킹 | frame 제어(X-Frame-Options/CSP) |
| **LDAP Injection** | LDAP 쿼리 조작으로 인증 우회·정보 노출 | 입력 이스케이프, 파라미터화 |
| **Mail Header Injection** | 메일 헤더에 개행 삽입 → 스팸/피싱 발송 | 개행 제거, 헤더 검증 |
| **OS Command Injection** | 입력이 셸로 전달되어 명령 실행 | 인자 배열 실행, 화이트리스트 |
| **PHP Code Injection** | `eval()` 등으로 PHP 코드 실행 | eval 사용 금지, 입력 검증 |
| **Server-Side Include (SSI) Injection** | SSI 지시문 삽입으로 서버측 실행 | SSI 비활성, 입력 필터 |

## 3. 공통 방어 원칙
- "데이터와 코드/명령의 경계"를 지키는 것이 모든 Injection 방어의 핵심 — **파라미터화·이스케이프·화이트리스트·최소권한**.
- 출력 컨텍스트(HTML/JS/URL/SQL/LDAP)에 맞는 인코딩을 적용.

## 4. 복습 질문
1. HTML Injection과 XSS의 차이는?
2. OS Command Injection을 코드 레벨에서 막는 방법은?
3. SSI Injection이 가능하려면 서버에 어떤 설정이 켜져 있어야 하나?
