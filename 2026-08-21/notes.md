# 2026-08-21 · SQL(DDL/DML) · HTTPS(SSL) · FTP(vsftpd) · DNS(named)

> 원본: [original/0821_260824_164919_1.pdf](original/0821_260824_164919_1.pdf)

## 1. SQL — 데이터베이스 생성과 접속
```
mysql -u root -p                    # root로 접속(-p: 비밀번호 입력)
CREATE DATABASE School;
USE School;                         # 사용할 DB 선택
```

### 테이블 생성 (DDL)
```
CREATE TABLE Student (
  Name  varchar(8),
  Date  int(4),
  Phone int(10)
);                                  # Student 테이블에 Name, Date, Phone 컬럼 생성
```
- `varchar(n)` 문자열, `int(n)` 정수.

### 테이블 구조 변경 — ALTER TABLE
| 키워드 | 역할 | 예시 |
|---|---|---|
| **add** | 컬럼 추가 | `ALTER TABLE Student add Address varchar(8);` |
| | 위치 지정 추가 | `... add Age int(3) after Date;` (Date 뒤에) |
| | 맨 앞 추가 | `... add MBTI varchar(4) first;` |
| | 여러 개 추가 | `... add (Age2 int(3), Age3 int(3));` |
| **modify** | 자료형·위치 수정 | `... modify Name varchar(8) first;` |
| **change** | 컬럼명+자료형 변경 | `... change MBTI Subject varchar(10);` |
| **drop** | 컬럼 삭제 | `... drop Age2;` |
- 정리: **add / change / modify / drop** 을 ALTER 뒤에 써서 구조를 바꾼다.

### 데이터 조작 (DML)
```
SELECT * FROM Student;              # 모든 컬럼 조회

INSERT INTO Student VALUES ('Kim','DB',2025,12345678,'Daegu',27);
                                    # 전체 컬럼 순서대로 값 삽입
INSERT INTO Student (Name,Date,Address) VALUES ('Hong',2026,'Busan');
                                    # 특정 컬럼만 지정해 삽입
```
- 지정하지 않은 컬럼은 **Null**이 허용돼 있지 않으면 에러 → 값이 반드시 필요.

```
UPDATE Student SET Date = 2026;                        # 모든 행의 Date를 2026으로
UPDATE Student SET Age = 30 WHERE Address = 'Busan';   # 조건에 맞는 행만 변경
UPDATE Student SET Name='Park', Date=2025 WHERE Age = 27;

DELETE FROM Student WHERE Name = 'Park';   # 조건에 맞는 행 삭제
DELETE FROM Student;                       # 모든 행 삭제(테이블 구조는 남음)
```
> **WHERE 절을 빼면 전체 행에 적용**되므로 UPDATE·DELETE 시 특히 조심.

### 테이블·DB 삭제
```
DROP TABLE Student;
DROP DATABASE School;
```
- `DELETE`는 데이터만 지우고, `DROP`은 구조 자체를 삭제한다.

## 2. HTTPS — SSL 인증서 (Apache)
HTTP에 SSL/TLS 암호화를 더한 것. **인증서·키 쌍**을 만들어 서버에 적용한다.
```
mkdir /etc/httpd/conf/ssl
cd /etc/httpd/conf/ssl

# 개인키 생성
openssl genrsa -des3 -out test.com.key

# CSR/자체서명 인증서 생성 (newkey rsa:2048, nodes=키 암호 없이)
openssl req -newkey rsa:2048 -nodes -keyout /etc/httpd/conf/ssl/test.com.key \
        -x509 -days 365 -out /etc/httpd/conf/ssl/test.com.crt

# 인증서 내용 확인
openssl x509 -noout -text -in test.com.crt
```

### httpd에 적용
```
vi /etc/httpd/conf.d/ssl.conf        # 인증서/키 저장 위치를 설정에 반영
systemctl restart httpd
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

## 3. FTP — 파일 전송 서버 (vsftpd)
**FTP = File Transfer Protocol.**
```
vi /etc/vsftpd/vsftpd.conf          # vsftpd 환경설정
systemctl enable vsftpd             # vsftpd 부팅 시 자동 실행
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload
```

### 테스트 계정·디렉터리
```
useradd ftptest                     # ftptest 유저 생성
passwd  ftptest                     # 비밀번호 설정 → 홈 /home/ftptest
cd /home/ftptest
mkdir document
mkdir music
touch ftptest.txt
```

### FTPS (FTP over SSL) 설정
```
# 인증서 생성
openssl req -newkey rsa:2048 -nodes -keyout /etc/vsftpd/vsftpd.pem \
        -x509 -days 360 -out /etc/vsftpd/vsftpd.pem

vi /etc/vsftpd/vsftpd.conf
```
```ini
ssl_enable=YES
rsa_cert_file=/etc/vsftpd/vsftpd.pem
force_local_logins_ssl=YES
ssl_tlsv1=YES
allow_anon_ssl=NO
pasv_enable=YES
pasv_min_port=50000
pasv_max_port=55000
```
```
firewall-cmd --permanent --add-port=50000-55000/tcp    # passive 포트 범위 개방
firewall-cmd --reload
systemctl restart vsftpd
```
- passive 모드용 포트 범위를 방화벽에서 함께 열어야 데이터 채널이 동작한다.

## 4. DNS — 네임 서버 (named / BIND)
도메인 ↔ IP를 변환하는 서버.
```
vi /etc/named.conf
```
```
# 외부 질의 허용 + zone 파일 경로를 /var/named 로
options {
    allow-query   { any; };
    recursion     yes;         # DNS 서버끼리 재귀 질의 허용
};
zone "dnstestskt.com" IN {
    type master;
    file "dnstestskt.com.zone";   # IP ↔ DNS 매핑이 담길 zone 파일
    allow-update { none; };
};
```
```
named-checkconf /etc/named.conf     # 설정 파일 문법 체크 (가능하면 이상 없음!)
cd /var/named
touch dnstestskt.com.zone           # zone 파일 생성 → 레코드 작성
```

## 5. 정정·보충
- 원본의 `alther table` → 올바른 SQL 키워드는 **`ALTER TABLE`**.
- `DELETE`는 WHERE로 행 단위 삭제(로그 남고 롤백 가능), `TRUNCATE`는 전체 행을 빠르게 비움, `DROP`은 테이블 자체 삭제 — 셋을 구분할 것.
- 자체서명(self-signed) 인증서는 실습용이며, 브라우저가 신뢰하지 않으므로 운영에서는 CA 발급 인증서를 쓴다.
- `-des3` 옵션으로 만든 키는 비밀번호가 걸려 서비스 재시작마다 입력을 요구 → 자동화하려면 `-nodes`로 생성.

## 6. 복습 질문
1. `DELETE`, `DROP`, `TRUNCATE`의 차이는?
2. UPDATE 문에서 WHERE를 빠뜨리면 어떤 결과가 되나?
3. FTP passive 모드에서 포트 범위를 방화벽에 열어야 하는 이유는?
4. DNS zone 파일은 무엇을 저장하는가?
