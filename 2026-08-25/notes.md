# 2026-08-25 · Windows 로컬 보안 정책 · NFS · Samba · rsyslog(+MariaDB)

> 원본: [original/0825_260825_154802_1.pdf](original/0825_260825_154802_1.pdf)

## 1. Windows — Local Security Policy
보안 설정을 변경할 수 있는 콘솔.
- **Account Policies**
  - **Password Policy**: 암호 복잡도·길이·만료 등
  - **Account Lockout Policy**: 로그인 실패 시 계정 잠금 (무차별 대입 방어)
- **Local Policies** (각 항목은 explain(설명) 페이지 참조해 적용/차단)
  - **Audit Policy**: 감사(로그 기록) 대상 설정
  - **User Rights Assignment**: 사용자 권한 할당
  - **Security Options**: 기타 보안 옵션

## 2. NFS (Network File System)
네트워크를 통해 **디렉터리(자원)를 공유**하는 시스템.
| 환경 | 방식 |
|---|---|
| Linux ↔ Linux | **NFS** |
| Windows ↔ Linux | **Samba** |

예시: `192.168.16.64`(NFS Server) ↔ `192.168.16.99`(NFS Client)

### Server
```
dnf install -y nfs-utils
systemctl enable nfs-server
systemctl enable rpcbind
mkdir /nfsserver                 # 공유할 디렉터리 생성
chmod 755 /nfsserver             # rwx 권한 부여

vi /etc/exports                  # 공유 대상·대역 설정
   /nfsserver 192.168.16.99(rw,sync,no_root_squash)
```
- `rw` 읽기·쓰기 허용, `sync` 변경 즉시 기록, **`no_root_squash`** = Client의 root를 Server의 root로 인정(주의: 보안상 위험할 수 있음).
```
exportfs -a                      # 적용(설정 반영)
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --reload
```

### Client
```
systemctl enable rpcbind
mkdir /nfsclient                 # 연결할 디렉터리 생성
mount -t nfs 192.168.16.64:/nfsserver /nfsclient
                                 # nfsclient dir ↔ nfsserver dir 연결(마운트)
```

## 3. Samba (Windows ↔ Linux 파일 공유)
- Linux = Server, Windows = Client.
- 포트: **smbd → tcp 445**, **nmbd → tcp 139, udp 137·138**

### Server 설정
```
dnf install -y samba samba-common samba-client
mkdir /sambatest
chmod 777 /sambatest             # Linux↔Windows 공유 디렉터리

useradd sambauser                # Samba용 사용자(리눅스 계정) 생성
smbpasswd -a sambauser           # Samba 사용자 비밀번호 설정
```
`smbpasswd` 옵션: `-a` add / `-d` disable / `-e` enable / `-x` 삭제 / `-n` 비밀번호 없이 사용

```
vi /etc/samba/smb.conf           # Samba 환경설정
```
```ini
[sambashare]                     ; 공유 디렉터리 이름(네트워크에 보이는 이름)
   path = /sambatest
   browsable = Yes
   writeable = Yes
   guest ok = Yes
   read only = No
   create mask = 0777            ; 생성되는 파일 rwx 권한
   directory mask = 0777         ; 생성되는 디렉터리 rwx 권한
   ; 비밀번호 없이 접속 허용하려면  null passwords = Yes  추가
```
```
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
systemctl start smb nmb

vi /etc/selinux/config
   SELINUX=disabled              # 커널/보안 설정, 바꾸면 재부팅 필요
```
> **SELinux**는 접근 통제 보안 모듈. 실습 편의상 disabled로 두지만, 운영에서는 끄기보다 정책(context)으로 허용하는 것이 바람직.

## 4. rsyslog — 중앙 로그 수집
Client에서 발생하는 로그를 **서버 한 곳에 모아** 확인하는 기능.

### Server
```
dnf install -y rsyslog rsyslog-doc
vi /etc/rsyslog.conf
```
```
# 저장 위치: /var/lib/rsyslog, 수신 포트 514
$ModLoad imudp
$InputUDPServerRun 514          # UDP 514 수신
$ModLoad imtcp
$InputTCPServerRun 514          # TCP 514 수신
```
```
firewall-cmd --permanent --add-port=514/udp
firewall-cmd --permanent --add-port=514/tcp
firewall-cmd --reload
systemctl start rsyslog
systemctl enable rsyslog
```

### Client
```
vi /etc/rsyslog.conf
   *.*  @@192.168.16.64:514       # @@ = TCP 전송
   *.*  @192.168.16.64:514        # @  = UDP 전송
```
- Client 로그가 Server의 `tail -f /var/log/messages`에서 실시간으로 올라오면 성공.

### (심화) 로그를 MariaDB에 저장
```
dnf install -y mariadb-server rsyslog-mysql

# rsyslog 제공 스키마로 DB 생성
mysql -u root -p < /usr/share/doc/rsyslog/mysql-createDB.sql
```
```sql
USE Syslog;
GRANT ALL PRIVILEGES ON Syslog.* TO 'rsyslog'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
```
```
vi /etc/rsyslog.conf
   $ModLoad ommysql
   *.*  :ommysql:127.0.0.1,Syslog,rsyslog,123456
                      └ db서버 IP ┘ └ DB,계정,PW ┘
```

## 5. 정정·보충
- `no_root_squash`는 클라이언트 root가 서버 파일을 root 권한으로 다루게 하므로 신뢰된 환경에서만. 기본값인 `root_squash`가 더 안전하다.
- Samba `create mask = 0777`은 실습용. 실제로는 최소 권한(예: 0664/0775)이 바람직.
- rsyslog 전송: `@` = UDP(빠르지만 유실 가능), `@@` = TCP(신뢰성). 중앙 로그 수집은 보통 TCP 권장.
- DB 계정 비밀번호를 설정 파일에 평문으로 두는 점은 보안상 약점 → 접근 권한·파일 퍼미션 관리 필요.

## 6. 복습 질문
1. NFS와 Samba는 각각 어떤 OS 조합에서 쓰는가?
2. `no_root_squash`가 보안상 위험한 이유는?
3. rsyslog에서 `@`와 `@@`의 차이는?
4. Account Lockout Policy는 어떤 공격을 막기 위한 것인가?
