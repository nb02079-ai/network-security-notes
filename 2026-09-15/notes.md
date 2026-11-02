# 2026-09-15 · 리눅스 로그 관리 · syslog/journalctl/logrotate · firewalld/iptables · BOF(ASLR)

> 원본: [original/Notes_260915_123120_1.pdf](original/Notes_260915_123120_1.pdf)

## 1. 로그 파일 (Rocky Linux, `/var/log`)
| 파일 | 내용 |
|---|---|
| **wtmp** | 성공한 로그인/로그아웃 정보를 담은 로그 |
| **secure** | 원격 접속·FTP 등 **인증 관련**을 거치는 모든 로그 기록 |
| **lastlog** | 사용자의 **최근 로그인** 기록 |
| **messages** | 시스템 전반 상태와 발생한 이벤트 기록 (Facility/Priority 기준으로 로그) |
| **btmp** | **실패한 로그인**만 기록하는 로그 (`lastb` 명령으로 확인) |
| **utmp** | **현재 로그인한** 사용자·상태 정보 |
| **xferlog** | FTP 전송 로그 기록 |
- `history`: 이전에 내린 **명령어** 기록 → `/root/.bash_history`(계정별)에 저장.
- `finger`: 사용자 계정의 상세 정보와 로그인 상태.

## 2. syslog — Facility / Priority
- 로그는 **Facility(출처)** 와 **Priority(심각도)** 로 분류.
- Priority 레벨(높음→낮음): **emerg → alert → crit → err → warning → notice → info → debug**

### 설정 — `/etc/rsyslog.conf` (구 `/etc/syslog.conf`)
```
*.crit    /var/log/critical.log
└Facility.Priority┘  └ 저장 위치 ┘
# 모든 Facility의 crit 이상 로그를 critical.log로
```

## 3. journalctl (systemd 저널)
```bash
journalctl -n 10      # 끝에서 10줄까지 표시
journalctl -f         # 실시간 표시(follow)
journalctl -r         # 역순 표시
```
- 재부팅 시 초기화되는 저널은 `/run/log/journal/.../system.journal`에 기록(영구 저장하려면 `/var/log/journal` 사용).

## 4. logrotate / crontab
- **logrotate**: 대량·큰 로그 파일이 디스크를 꽉 채우지 않도록 **백업/압축/순환**. 해당 주기 설정.
- **crontab**: 작업 스케줄링(정해진 시간에 명령 자동 실행).

## 5. 리눅스 기재 방화벽 — firewalld
```bash
firewall-cmd --get-zones            # 적용 가능한 zone 목록
firewall-cmd --get-services         # 현재 zone에 허용된 서비스 목록
firewall-cmd --get-active-zones     # 현재 활성화된 zone 목록
firewall-cmd --list-all --zone=<zone>     # 해당 zone의 기본 방화벽 규칙 확인
firewall-cmd --set-default-zone=<zone>    # 기본 zone 설정

# rich rule (세밀한 규칙)
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.16.34" reject'
   # ipv4 192.168.16.34를 거절하는 rich-rule
firewall-cmd --reload
```

## 6. iptables
chain과 table을 사용해 규칙을 세밀히 조정 가능.
```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   # 외부에서 들어오는 tcp(80) 허용
iptables -A INPUT -s 192.168.16.34 -j DROP
   # 192.168.16.34가 source인 모든 통신 DROP
iptables -D INPUT ...      # 해당 rule 삭제(-D = delete)
```
- 주요 chain: INPUT(들어옴)/OUTPUT(나감)/FORWARD(경유), target: ACCEPT/DROP/REJECT.

## 7. BOF (Buffer Overflow) — 실습용 보호기법 비활성화
버퍼 오버플로 실습 시, 리눅스 메모리 보호기법 **ASLR**(주소 공간 랜덤화)을 꺼서 재현성을 높인다.
```bash
sudo sysctl -w kernel.randomize_va_space=0
# 또는
echo 0 > /proc/sys/kernel/randomize_va_space
```
> ⚠️ ASLR 비활성화는 **격리된 실습 환경에서 공격 원리를 학습**하기 위한 것. 운영 시스템에서는 절대 끄지 않는다 — ASLR·NX·스택카나리·PIE는 중요한 완화책이다.

## 8. 정정·보충
- `secure`는 데비안 계열에선 `/var/log/auth.log`에 해당.
- iptables의 후속/대체는 **nftables**(`nft`). 최신 배포판은 firewalld가 내부적으로 nftables를 쓰기도 한다.
- journalctl에서 특정 서비스만: `journalctl -u sshd`.

## 9. 복습 질문
1. wtmp·btmp·lastlog는 각각 무엇을 기록하나?
2. syslog Priority에서 `crit`은 `warning`보다 심각한가?
3. `firewall-cmd`에서 `--permanent` 없이 추가하면?
4. BOF 실습 때 ASLR을 끄는 이유와, 운영에서 끄면 안 되는 이유는?
