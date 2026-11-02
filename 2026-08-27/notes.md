# 2026-08-27 · SQL 조회·집계·백업 · DB 복제(Replication) · Ubuntu/Docker · PMM 모니터링

> 원본: [original/0827_260828_153741_1.pdf](original/0827_260828_153741_1.pdf)

## 1. MariaDB — 테이블 생성과 조회 심화
```sql
CREATE DATABASE library;
USE library;
CREATE TABLE books (
  id             int auto_increment primary key,   -- 자동 증가 기본키
  title          varchar(100) not null,            -- 반드시 값 필요
  author         varchar(50),
  published_year int,
  available      boolean default true              -- 미지정 시 true
);

INSERT INTO books (title, author, published_year, available)
VALUES ('소설책', '제인도우 작가', 1919, true);
```

### 조건·정렬·범위 조회
```sql
SELECT * FROM books WHERE published_year >= 2000;
SELECT * FROM books WHERE author = 'Jildong' AND available = true;   -- 두 조건 모두 만족
SELECT * FROM books ORDER BY published_year ASC;     -- 오름차순(내림차순은 DESC)
SELECT * FROM books ORDER BY published_year DESC LIMIT 1;  -- 내림차순 후 1개만(최신 1건)
SELECT * FROM books WHERE published_year BETWEEN 1900 AND 2000;  -- 1900~2000 범위
SELECT * FROM books WHERE title LIKE '%data%';       -- title에 data 포함
SELECT * FROM books WHERE title IS NOT NULL;         -- title이 비어있지 않은 행
```

### 집계 함수
```sql
SELECT count(*) AS total_books FROM books;              -- 행 개수
SELECT sum(published_year) AS total_year FROM books;    -- 합계
SELECT avg(published_year) AS avg_year,
       sum(published_year) AS total_year FROM books;    -- 여러 집계 동시
```
| 함수 | 의미 |
|---|---|
| `sum` | 합계 |
| `avg` | 평균 |
| `min` | 최소 |
| `max` | 최대 |
| `count` | 개수 |
- `AS`로 결과 컬럼에 **별칭(alias)** 을 붙인다.

## 2. 데이터베이스 백업과 복원
백업은 `mysqldump`(셸 명령), 복원은 리다이렉션으로.
```bash
# 백업 (export)  >
mysqldump -u root -p library > backup.sql              # library DB 백업
mysqldump -u root -p --all-databases > all_backup.sql  # 모든 DB 백업
mysqldump -u root -p library books > books_backup.sql  # library의 books 테이블만

# 복원 (import)  <
mysql -u root -p library < back.sql                    # back.sql을 library로 복원
```
- 방향 기억: **`>` 내보내기(백업), `<` 불러오기(복원).**

## 3. DB 복제 (Replication) — Master ↔ Slave
Master DB의 변경을 Slave 서버로 복제.

### DB 복제 방식
- **비동기(Async)**: Master가 기록 후 바로 응답 — MySQL **기본값**
- **반동기(Semi-sync)**: 최소 1개 Slave 수신 확인 후 응답
- **동기(Sync)**: 모든 Slave 반영 후 응답

### Master 서버
```
vi /etc/my.cnf.d/mariadb-server.cnf
```
```ini
[mysqld]
server-id = 1                              # 고유 식별자(유니크)
log_bin = /var/log/mariadb/mysql-bin.log   # 이진 로그 파일(변경사항 기록)
expire_logs_days = 10                      # 이진 로그 유지 기간
max_binlog_size = 100M                     # 이진 로그 용량 제한(초과 시 새 파일)
```
```sql
-- 복제 전용 계정 생성
GRANT REPLICATION SLAVE ON *.* TO 'slave_db'@'%' IDENTIFIED BY 'slave_password';
FLUSH PRIVILEGES;
```

### Slave 서버
```ini
[mysqld]
server-id = 2        # Master와 다른 값
```
```sql
CHANGE MASTER TO
  master_host     = '192.168.16.119',
  master_user     = 'slave_db',
  master_password = 'slave_password',
  master_port     = 3306,
  master_log_file = 'mysql-bin.000002',   -- Master의 현재 로그 파일
  master_log_pos  = 125165;               -- Master의 현재 로그 위치
START SLAVE;
```
- `master_log_file`·`master_log_pos`는 Master에서 `SHOW MASTER STATUS`로 확인한 값을 넣는다.

## 4. Ubuntu — DB 서버 & Docker 설치
Rocky(dnf) 대신 Ubuntu(apt) 환경.
```bash
sudo apt install -y iputils-ping       # ping 없을 때 설치
sudo apt install -y vim
sudo vi /etc/netplan/50-cloud-init.yaml # 네트워크 설정(Ubuntu는 netplan)
sudo netplan apply
```

### Docker 설치 (공식 저장소)
```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings/            # key 디렉터리 권한 설정
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
     | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg           # 읽기 권한
# 저장소 등록 (dpkg --print-architecture, os-release 코드네임 사용)
sudo apt install -y docker-ce docker-ce-cli containerd.io \
     docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker                    # 부팅 시 자동 실행 + 지금 시작
```

## 5. PMM (Percona Monitoring and Management)
DB 성능·상태를 모니터링하는 도구. Server(컨테이너)와 Client로 구성.

### PMM Server (Docker)
```bash
sudo curl -fsSL https://www.percona.com/get/pmm | sudo /bin/bash
   # PMM server 설치 및 docker 실행,  기본 username: admin / pw: admin
# https://192.168.16.65 로 PMM 웹 접속
```

### PMM Client
```bash
sudo wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
sudo dpkg -i percona-release_latest.generic_all.deb
sudo percona-release enable pmm3-client
sudo apt update
sudo apt install -y pmm-client
sudo pmm-admin config --server-insecure-tls \
     --server-url=https://admin:admin@192.168.16.65:443   # PMM 서버와 연동
sudo pmm-admin status                                      # 연동 확인

# 모니터링 대상 MariaDB 준비
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb-server
sudo mysql_secure_installation
sudo mysql -u root -p
```
```sql
-- PMM 전용 모니터링 계정
CREATE USER 'pmm'@'localhost' IDENTIFIED BY 'pmm1234' WITH max_user_connections 10;
GRANT SELECT, PROCESS, REPLICATION CLIENT, RELOAD ON *.* TO 'pmm'@'localhost';
FLUSH PRIVILEGES;
```
```bash
# MariaDB를 PMM 모니터링에 등록
sudo pmm-admin add mysql --username=pmm --password=pmm1234 \
     --host=localhost --port=3306 --query-source=slowlog \
     --environment=production
```

## 6. 정정·보충
- `boolean`은 MariaDB에서 내부적으로 `tinyint(1)`로 저장된다.
- 복제 계정 권한은 **최소 권한 원칙**상 `REPLICATION SLAVE`만 주는 것이 맞다 (PMM 계정도 SELECT/PROCESS 등 조회 권한만).
- PMM의 기본 `admin/admin`은 최초 접속 후 반드시 변경.
- Docker 설치 시 `curl`로 docker.com GPG 키를 받는 단계가 있는데, 실습 환경에 따라 외부 접근이 막히면 사내 미러를 쓰기도 한다.

## 7. 복습 질문
1. `>`와 `<`는 백업/복원에서 각각 어떤 방향인가?
2. 비동기·반동기·동기 복제의 차이는?
3. Replication에서 Master와 Slave의 `server-id`는 왜 달라야 하나?
4. `ORDER BY ... DESC LIMIT 1`은 어떤 결과를 주는가?
