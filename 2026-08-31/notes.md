# 2026-08-31 · Docker 기초 · Zabbix(NMS) 구축 · GNS3 Reflexive/Dynamic ACL

> 원본: [original/0831_260831_170949_1.pdf](original/0831_260831_170949_1.pdf)

## 1. Docker 기초 명령
```bash
sudo docker pull ubuntu:bionic          # 이미지 받기(태그 생략 시 latest)
sudo docker run -it ubuntu:bionic bash  # 컨테이너 실행 후 bash 진입(-it 대화형)
   # 컨테이너 안에서
   apt update -y
   apt install -y git

sudo docker ps          # 실행 중 컨테이너 목록 (-a: 중지된 것 포함)
sudo docker rmi <이미지>  # 이미지 삭제
sudo docker rm <컨테이너> # 컨테이너 삭제
```

## 2. 네트워크 보안 — NMS(Network Management System)
여러 장비(PC·노트북·IoT 등)를 **중앙 서버에서 모니터링·관리**하는 시스템.
- 대표 솔루션: **Zabbix**, LogicMonitor 등
- NMS 서버에서 HW/SW/IP 상태를 한눈에 보고 관리 가능.

### Zabbix 서버 구축 (Ubuntu, 예: 192.168.16.136)
Zabbix는 웹 UI가 있어 **Web 서버 + DB 서버 + PHP**가 함께 필요하다 (LAMP 유사).
```bash
sudo apt install -y apache2 php php-fpm mariadb-server
# Zabbix 공식 저장소 등록
sudo wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb   # 저장소 설치
sudo apt update                                          # 저장소 변경 반영

sudo apt install -y zabbix-server-mysql zabbix-frontend-php \
     zabbix-apache-conf zabbix-sql-scripts zabbix-agent2 \
     php-mysql php-gd php-bcmath php-net-socket
```

### DB 준비
```bash
sudo mysql_secure_installation
```
```sql
CREATE DATABASE my_zabbix_db CHARACTER SET utf8 COLLATE utf8_bin;  -- 한글 가능 DB
CREATE USER 'zabbix_user'@'localhost' IDENTIFIED BY '1234';
GRANT ALL PRIVILEGES ON my_zabbix_db.* TO 'zabbix_user'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
```bash
# Zabbix 제공 초기 스키마를 DB에 적재
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | \
     mysql --default-character-set=utf8mb4 -u zabbix_user -p'1234' my_zabbix_db
```

### 설정 파일 수정
```bash
sudo vi /etc/zabbix/zabbix_server.conf     # DBName, DBUser, DBPassword 변경
sudo vi /etc/zabbix/zabbix_agent2.conf     # 모니터링 대상(자기 자신) 설정
sudo systemctl enable --now apache2 zabbix-server zabbix-agent2
```

### PHP 튜닝 (Zabbix 요구값) — `/etc/php/8.3/fpm/pool.d/www.conf`
```ini
php_value[max_execution_time] = 300
php_value[memory_limit] = 128M
php_value[post_max_size] = 16M
php_value[upload_max_filesize] = 2M
php_value[max_input_time] = 300
php_value[max_input_vars] = 10000
php_value[always_populate_raw_post_data] = -1
php_value[date.timezone] = Asia/Seoul
```
```bash
sudo systemctl restart apache2 php8.3-fpm
# http://192.168.16.136/zabbix 로 접속해 웹 설치 마법사 진행
```

### 방화벽 + Agent
```bash
sudo apt install -y firewalld
sudo firewall-cmd --permanent --add-port=10051/tcp   # Zabbix 서버 포트
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

### Zabbix Client (Agent, 예: 192.168.16.127)
```bash
sudo apt install -y zabbix-agent2
sudo vi /etc/zabbix/zabbix_agent2.conf
   # Server, ServerActive → 서버 IP로 변경
   # Hostname = zabbix_ubuntu_test
sudo apt install -y firewalld
sudo firewall-cmd --permanent --add-port=10050/tcp   # Agent 포트
sudo firewall-cmd --reload
# Zabbix 웹에서 Host: zabbix_ubuntu_test 추가 → 모니터링 시작
```
- 포트 정리: **10051 = 서버**, **10050 = 에이전트**.

## 3. GNS3 — Reflexive ACL / Dynamic ACL
토폴로지: 새부(내부) R1–R2 … R3–R4(외부), R2의 fo/1 ↔ R3 연결.

| 약어 | 의미 | 성격 |
|---|---|---|
| **RACL** | Reflexive ACL | 내부 |
| **DACL** | Dynamic ACL (Lock-and-Key) | 외부 |

### Reflexive ACL (RACL) — 내부에서 시작한 연결만 복귀 허용
- 내부에서 시작한 TCP·UDP·ICMP는 허용하고, **외부에서 먼저 들어오는 연결은 모두 차단**.
- R2에서 외부와 연결된 인터페이스의 **out bound**(나가는) 방향에 정책 설정.
```
conf t
ip access-list extended racl
   permit tcp any any reflect racltest     ! 나가는 트래픽을 racltest로 기록
   permit udp any any reflect racltest
   permit icmp any any reflect racltest
   permit ip any any

ip access-list extended test                ! 들어오는 트래픽 검사
   permit ospf host 10.10.20.3 any          ! 라우팅 프로토콜은 통과
   evaluate racltest                         ! 기록된 세션(나갔던 연결)의 복귀만 허용

interface Gi0/1
   ip access-group racl out                  ! 나갈 때 racl
   ip access-group test in                   ! 들어올 때 test
```
- 핵심: **내부에서 시작한 연결의 "반대 방향" 복귀 트래픽만** 임시로 거슬러 허용 → 외부 선제 접속 차단.

### Dynamic ACL (DACL, Lock-and-Key)
- 사용자가 라우터에 **인증(telnet/ssh)** 에 성공하면, 그때 **임시로 통로가 열려** 이후 통신 가능.
- 인증 전에는 막혀 있다가 인증 후 동적으로 permit이 생성되는 방식.

## 4. 정정·보충
- Reflexive ACL은 **Extended Named ACL**에서만 쓸 수 있고(`reflect`/`evaluate`), 세션 상태를 임시 기록한다 — 단순 stateless ACL보다 stateful에 가깝다.
- Zabbix 포트 혼동 주의: Agent passive는 서버→에이전트(10050), active는 에이전트→서버(10051).
- Docker `rm`은 컨테이너, `rmi`는 이미지. 실행 중 컨테이너는 `-f`로 강제 삭제.

## 5. 복습 질문
1. Reflexive ACL이 "내부에서 시작한 연결만 허용"하는 원리를 한 문장으로?
2. Dynamic ACL이 일반 ACL과 다른 결정적 특징은?
3. Zabbix에서 10050과 10051 포트는 각각 누구의 포트인가?
4. `docker run -it`의 `-it`는 무엇을 의미하나?
