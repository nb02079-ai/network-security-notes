# 2026-08-24 · DNS zone 파일 · Windows Server(IIS·FTP·DNS)

> 원본: [original/0824_260824_154346_1.pdf](original/0824_260824_154346_1.pdf)

## 1. Linux DNS — zone 파일 작성 (이어서)
08-21에서 만든 zone 파일에 실제 레코드를 작성한다.
```
vi /var/named/dnstestskt.com.zone
```
```dns
$TTL 86400
@   IN  SOA  ns.dnstestskt.com. root.dnstestskt.com. (
                20260824     ; Serial (갱신 번호, 보통 날짜)
                24H          ; Refresh
                15M          ; Retry
                48H          ; Expire
                86400 )      ; Minimum TTL
    IN  NS   ns.dnstestskt.com.
    IN  A    192.168.16.56
ns  IN  A    192.168.16.56
www IN  A    192.168.16.56
```
- **SOA**: 해당 zone의 권한 시작(Start Of Authority) 레코드 — 기준 네임서버·관리자·갱신 주기.
- **NS**: 이 도메인을 담당하는 네임서버.
- **A**: 도메인(호스트) → **IPv4** 주소 매핑. (`ns`, `www` 같은 호스트별로 지정)

### 서비스 적용
```
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload
systemctl start named
```

### 클라이언트에서 확인하는 3가지 경로
- **hosts 파일**: 로컬에 직접 등록한 매핑 우선 확인
- **local DNS**: 로컬 DNS 서버 질의
- **zone 파일**: `ftp.dnstestskt.com = 192.168.16.56` 처럼 연결됨을 확인

## 2. Windows Server 설정 — Server Manager
리눅스에 이어 **Windows Server**로 같은 서비스(웹·FTP·DNS)를 구성한다.
- **Add Roles and Features**: 원하는 서비스(역할)를 선택해 설치.

### IIS (Internet Information Services) — 웹 서버
Add Website 후:
| 항목 | 역할 |
|---|---|
| **Default Document** | 사이트 접속 시 기본으로 열 파일의 **우선순위** 지정 (예: index.html) |
| **Logging** | 로그 관련 설정 |
| **MIME Types** | 확장자별 파일 처리 방식(형식) 관리 |
- 문서 루트: `Local disk C → inetpub → wwwroot` 안에 html 파일을 둔다.

### Computer Management — 사용자/그룹
- Local Users and Groups → **Users**에서 이용자(계정) 생성, **group** 설정.

### IIS — FTP 서버
| 항목 | 역할 |
|---|---|
| **Add FTP Server** | FTP 사이트 추가 |
| **Physical path** | `inetpub\ftproot` |
| **Authorization** | 접근 가능 인원·권한 설정 |
| **FTP Current Sessions** | 현재 접속 목록 |
| **FTP Directory Browsing** | 디렉터리 탐색 허용 여부 |
| **FTP IP Address and Domain Restrictions** | IP별 접근 허용/차단 |
| **FTP Request Filtering** | 확장자별 요청 허용/차단 |
- SSL 적용 시, Windows Defender Firewall에서 해당 포트를 열어야(방화벽에 추가) 한다.
- FTP 로그 위치: `C:\inetpub\logs\LogFiles\FTPSVC2`

## 3. Windows Server — DNS
Forward Lookup Zones에서 새 zone 생성 (예: `dnstestws.com`) → SOA·NS 수동 설정.

| 항목 | 역할 |
|---|---|
| **Name Servers** | IP Address 적용 |
| **New Host (A)** | 연결할 IP 설정 |
| **Pointer(PTR) record** | 역방향(Reverse) zone 생성 시 **IP → 도메인** 변환 정보 저장 |
| **Alias(CNAME)** | 별명 설정 — 예: `myweb.dnstestws.com → www.dnstestws.com` |

- **정방향(Forward) = 도메인→IP**, **역방향(Reverse, PTR) = IP→도메인**.
- **CNAME(Alias)**: 한 호스트에 여러 이름을 붙일 때.

## 4. 진행 메모 (원본 하단)
- 21일 → HTTPS, 24일 → FTPS, 25일 → FTP(Windows) — 날짜별 실습 주제 메모.

## 5. 정정·보충
- SOA의 Serial은 zone을 수정할 때마다 **증가**시켜야 secondary 서버가 갱신을 인식한다 (날짜+순번 `YYYYMMDDnn` 관례).
- A 레코드는 IPv4, **AAAA 레코드는 IPv6** 매핑.
- Windows의 Alias는 DNS 표준의 **CNAME** 레코드에 해당.

## 6. 복습 질문
1. 정방향 zone과 역방향(PTR) zone의 차이는?
2. SOA 레코드의 Serial 값을 올려야 하는 이유는?
3. A 레코드와 CNAME 레코드는 각각 무엇을 매핑하는가?
4. IIS에서 Default Document는 무엇을 정하는가?
