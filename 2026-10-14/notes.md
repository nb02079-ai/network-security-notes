# 2026-10-14 · bWAPP(Bee-Box) ③ — TLS/암호 공격 · 경로/파일 · SSRF/XXE

> 원본: [original/Notes_261014_091554_1.pdf](original/Notes_261014_091554_1.pdf)
>
> ⚠️ bWAPP/Bee-Box 실습. 취약점 카탈로그 + 방어. 실제 시스템 대상 사용 금지.

## 1. 인코딩
- **Base64 encoding (secret)**: secret 값이 Base64로만 인코딩되어 있으면 쉽게 디코딩 → 인코딩은 암호화가 아님.

## 2. TLS/SSL 관련 공격
| 공격 | 핵심 |
|---|---|
| **BEAST** | TLS 1.0 이하에서 가능 / 0-day 성격, 전용 도구로 검증 |
| **CRIME** | TLS 압축을 악용한 정보 노출 |
| **BREACH** | HTTP 응답 압축 악용 |
| **Heartbleed** | OpenSSL heartbeat 버그로 메모리 내용(키·세션) 유출 (CVE-2014-0160) |
| **Clear-text HTTP (Credentials)** | 평문 전송된 자격증명 탈취 |
- 방어: **TLS 1.2+ 사용, 압축 비활성(CRIME/BREACH), OpenSSL 패치(Heartbleed), 전구간 HTTPS**.

## 3. Host Header 공격
| 변형 | 설명 |
|---|---|
| **Reset Poisoning** | 비밀번호 재설정 링크에 조작된 Host가 들어가 피싱 링크 생성 |
| **Cache Poisoning** | 조작된 Host가 캐시에 저장되어 다른 사용자에게 제공 |
- 방어: Host 헤더 화이트리스트, 절대 URL을 신뢰된 설정값으로 생성.

## 4. 접근제어 / 경로 취약점
| 유형 | 설명 | 방어 |
|---|---|---|
| **Missing Functional Level Access Control** | 기능 단위 인가 누락(숨은 관리 기능 직접 호출) | 서버측 기능별 인가 |
| **Directory Traversal (Directories/Files)** | `../`로 상위 경로 접근 | 경로 정규화, 화이트리스트 |
| **SQLite Manager Local File Inclusion** | LFI로 로컬 파일 포함 | 입력 검증, 포함 경로 제한 |
| **Remote & Local File Inclusion (RFI/LFI)** | 원격/로컬 파일을 include → 코드 실행/파일 노출 | `allow_url_include=Off`, 화이트리스트 |
| **Restrict Device/Folder Access** | 디바이스·폴더 접근 제한 미흡 | 권한·경로 제한 |

## 5. SSRF / XXE
| 유형 | 설명 | 방어 |
|---|---|---|
| **SSRF (Server-Side Request Forgery)** | 서버가 공격자 지정 내부 주소로 요청하게 유도(내부망·메타데이터 접근) | 아웃바운드 화이트리스트, 내부대역 차단 |
| **XXE (XML External Entity)** | XML 외부 엔티티로 파일 읽기·SSRF | 외부 엔티티·DTD 비활성 |

## 6. 복습 질문
1. "인코딩 ≠ 암호화"를 Base64 secret 예로 설명하면?
2. Heartbleed가 위험한 이유(무엇이 유출되나)는?
3. Host Header로 비밀번호 재설정 피싱이 가능한 흐름은?
4. XXE를 막는 핵심 설정은?
