# 2026-10-19 · bWAPP(Bee-Box) ④ — 알려진 CVE·웹 취약점 총정리 · Suninatas

> 원본: [original/Notes_261019_093105_1.pdf](original/Notes_261019_093105_1.pdf) (오전) · [original/Notes_261019_103524_1.pdf](original/Notes_261019_103524_1.pdf) (오후)
>
> ⚠️ bWAPP/Bee-Box 실습 마무리 + 워게임 사이트 메모. 실제 시스템 대상 사용 금지.

## 1. 알려진 애플리케이션 취약점 (CVE 계열)
| 유형 | 설명 |
|---|---|
| **Buffer Overflow (Local/Remote)** | 버퍼 경계를 넘겨 제어 흐름 탈취 |
| **Drupal SQL Injection (Drupageddon)** | CVE-2014-3704, 인증 전 SQLi |
| **Heartbleed Vulnerability** | OpenSSL 메모리 유출(CVE-2014-0160) |
| **PHP CGI Remote Code Execution** | CVE-2012-1823, php-cgi 인자 주입 |
| **PHP Eval Function** | `eval()` 악용 코드 실행 |
| **phpMyAdmin BBCode Tag XSS** | 알려진 XSS |
| **Shellshock Vulnerability (CGI)** | Bash 환경변수 파싱 버그(CVE-2014-6271), CGI 경유 RCE |
| **SQLite Manager** (LFI / PHP Code Injection / XSS) | 구성요소 취약점 |
- 공통 교훈: **버전 패치**가 최우선. 알려진 CVE는 공개 PoC로 바로 악용됨.

## 2. 웹 로직/클라이언트 취약점
| 유형 | 설명 | 방어 |
|---|---|---|
| **Unvalidated Redirects & Forwards** | 검증 없는 리다이렉트 → 피싱 | 대상 URL 화이트리스트 |
| **Clickjacking** | 투명 iframe으로 클릭 가로채기 | X-Frame-Options/CSP frame-ancestors |
| **Client-Side Validation (Password)** | 검증을 클라이언트에만 의존 | 서버측 검증 필수 |
| **HTTP Parameter Pollution** | 중복 파라미터로 로직 혼란 | 파라미터 정규화 |
| **HTTP Response Splitting** | 응답에 개행 삽입 | 헤더 개행 제거 |
| **HTTP Verb Tampering** | 허용 메서드 우회 | 메서드별 인가 |
| **Information Disclosure** (Favicon/Headers/PHP version/Robots File) | 버전·경로 노출 | 배너·헤더 최소화 |
| **Insecure iFrame (Login-Form)** | iframe 내 로그인 폼 | frame 제어 |
| **Unrestricted File Upload** | 실행 파일 업로드 | 확장자·내용·실행권한 검증 |

## 3. 학습 플랫폼 메모
- **WebGoat** → HTTP basics: 프록시로 요청 확인, Access Control Matrix, AJAX Security, Authentication flaws 등 단계별 학습.
- **Mutillidae** 류와 함께 OWASP 취약점 전반을 실습.

## 4. Suninatas (워게임)
- 오후 필기는 **Suninatas**(국내 웹 보안 워게임 사이트) 제목만 기록 — 이후 문제 풀이 실습으로 이어지는 시작 지점.

## 5. 복습 질문
1. Shellshock가 CGI를 통해 RCE로 이어지는 원리는?
2. Unvalidated Redirect가 피싱에 악용되는 방식과 방어는?
3. Clickjacking을 막는 헤더/정책은?
4. 클라이언트 측 검증만으로 불충분한 이유는?

---

> 이 파일로 업로드해주신 네트워크/정보보안 필기 정리가 마무리됩니다.
