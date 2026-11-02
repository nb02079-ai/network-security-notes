# 네트워크 · 정보보안 학습 정리

네트워크·정보보안 교육 과정에서 직접 필기한 내용을 **원본 스캔 + 학습 정리본**으로 날짜별로 모아둔 저장소입니다.
각 날짜 폴더에는 손필기 원본 PDF(`original/`)와, 그 내용을 다시 정리하고 보충·정정한 `notes.md`가 함께 있습니다.

## 구성

```
2026-MM-DD/
├── original/     원본 손필기 스캔 PDF
└── notes.md      핵심 개념 정리 + 명령어/설정 예시 + 정정·보충 + 복습 질문
```

## 날짜별 목차

| 날짜 | 주제 | 정리 |
|---|---|---|
| 2026-08-12 | 정보보안 개론 · OSI/TCP-IP · IP 주소 · Packet Tracer · 라우터 기본 설정 | [notes](2026-08-12/notes.md) |
| 2026-08-13 | 서브네팅(VLSM) · 정적 라우팅 · RIP | [notes](2026-08-13/notes.md) |
| 2026-08-14 | EIGRP · 재분배 · VLAN · Router-on-a-stick · DHCP | [notes](2026-08-14/notes.md) |
| 2026-08-18 | OSPF · 프로토콜 간 재분배 · ACL · NAT/PAT · VPN 개요 | [notes](2026-08-18/notes.md) |
| 2026-08-19 | VPN(GRE·IPsec) · FHRP(HSRP) · IPv6 · L3/L2 스위치 | [notes](2026-08-19/notes.md) |
| 2026-08-20 | 시스템 보안 입문 · VirtualBox · Linux 기초 · 권한 · APM/웹서버 | [notes](2026-08-20/notes.md) |
| 2026-08-21 | SQL(DDL/DML) · HTTPS(SSL) · FTP(vsftpd) · DNS(named) | [notes](2026-08-21/notes.md) |
| 2026-08-24 | DNS zone 파일 · Windows Server(IIS·FTP·DNS) | [notes](2026-08-24/notes.md) |
| 2026-08-25 | Windows 로컬 보안 정책 · NFS · Samba · rsyslog(+MariaDB) | [notes](2026-08-25/notes.md) |
| 2026-08-27 | SQL 조회·집계·백업 · DB 복제 · Ubuntu/Docker · PMM 모니터링 | [notes](2026-08-27/notes.md) |
| 2026-08-28 | MariaDB 권한 관리(GRANT/REVOKE) | [notes](2026-08-28/notes.md) |
| 2026-08-31 | Docker 기초 · Zabbix(NMS) 구축 · Reflexive/Dynamic ACL | [notes](2026-08-31/notes.md) |
| 2026-09-03 | ASAv Failover(HA) · pfSense · 공격용 webterm 설정 | [notes](2026-09-03/notes.md) |
| 2026-09-07 | pfSense 라우팅·방화벽 규칙(GNS3 연동) | [notes](2026-09-07/notes.md) |
| 2026-09-09 | IDS/IPS · DDoS · hping3(패킷 생성/플러딩) | [notes](2026-09-09/notes.md) |
| 2026-09-10 | SOC · SIEM/XDR/SOAR · Wazuh | [notes](2026-09-10/notes.md) |
| 2026-09-14 | 클라우드 컴퓨팅 기초(IaaS/PaaS/SaaS·Docker/K8s·MSA) | [notes](2026-09-14/notes.md) |
| 2026-09-15 | 리눅스 로그 관리 · syslog/journalctl/logrotate · firewalld/iptables · BOF | [notes](2026-09-15/notes.md) |
| 2026-09-17 | 모의해킹 방법론 · 정찰 · LFI · 권한상승 · redis CTF | [notes](2026-09-17/notes.md) |
| 2026-09-18 | 웹 정찰 짧은 메모(gobuster) | [notes](2026-09-18/notes.md) |
| 2026-09-21 | enum4linux · 리버스 셸 · getcap/setcap 권한상승 · DVWA Brute Force | [notes](2026-09-21/notes.md) |
| 2026-09-22 | 정찰 복습 · SSH키 크랙 · linPEAS · CVE 권한상승 · DVWA Command Injection | [notes](2026-09-22/notes.md) |
| 2026-10-02 | CTF 모의해킹 공격 체인(Double Trouble/inner) — 방어 관점 정리 | [notes](2026-10-02/notes.md) |

## 주제 인덱스

- **라우팅/스위칭**: OSI·TCP/IP(08-12), 서브네팅·RIP(08-13), EIGRP·VLAN·DHCP(08-14), OSPF·재분배(08-18), FHRP·IPv6·L3 스위치(08-19)
- **네트워크 보안**: ACL(08-18), NAT/PAT(08-18), VPN GRE·IPsec(08-18~19), Reflexive/Dynamic ACL(08-31), ASA Failover·pfSense·UTM(09-03·09-07)
- **시스템/서버**: Linux 기초·권한·APM(08-20), HTTPS·FTP·DNS(08-21), Windows Server IIS/FTP/DNS(08-24), 로컬 보안 정책·NFS·Samba·rsyslog(08-25), 로그 관리·firewalld/iptables·BOF(09-15)
- **데이터베이스**: SQL DDL/DML(08-21), 조회·집계·백업·복제(08-27), 권한 관리(08-28)
- **운영/모니터링**: Ubuntu·Docker·PMM(08-27), Docker·Zabbix(08-31), SOC·SIEM/XDR/SOAR·Wazuh(09-10)
- **클라우드**: IaaS/PaaS/SaaS·Docker/K8s·MSA(09-14)
- **방어 탐지**: IDS/IPS·DDoS(09-09)
- **모의해킹/CTF**: hping3(09-09), 정찰·LFI·권한상승·redis(09-17), gobuster(09-18), enum4linux·리버스셸·capability·Brute Force(09-21), john·linPEAS·CVE·Command Injection(09-22), 공격 체인 + 방어 매핑(10-02)

## 별도 폴더

- [`_career/`](_career/) — 같은 노트에 섞여 있던 자기소개서·포트폴리오·면접 준비 메모(기술 필기와 성격이 달라 분리).
- [`_ai/`](_ai/) — 머신러닝/논문 스터디 메모(앙상블·Random Subspace 등).

---

> 정리본의 "정정·보충" 항목은 원본 필기의 오기나 생략된 부분을 바로잡고 실무 맥락을 덧붙인 것이며,
> "복습 질문"은 스스로 점검용으로 추가한 것입니다.
