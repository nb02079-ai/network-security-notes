# 2026-08-20 · 시스템 보안 입문 · VirtualBox · Linux 기초 · 권한 · APM/웹서버

> 원본: [original/0820_260820_153644.pdf](original/0820_260820_153644.pdf)

여기서부터 네트워크 장비 설정을 넘어 **시스템(서버) 보안** 영역으로 넘어간다.

## 1. Oracle VirtualBox — VM 생성 설정
- **Processor**: Nested VT-x / AMD-V 기능 활성화 (VM 안에서 또 가상화를 돌릴 때 필요)
- **저장소**: 부팅할 **ISO 파일** 선택
- **네트워크**: **NAT** = 사설 IP를 공유기처럼 공인으로 변환해 VM이 외부와 통신
  - 어댑터에 브리지(Bridged): host 네트워크 대역 중 남는 IP를 VM이 직접 할당받음
  - **Promiscuous mode(무차별 모드)**: 자기 앞으로 오는 패킷이 아니어도 모두 수신 → 패킷 캡처·스니핑 실습에 사용

### VM 네트워크 확인
- `ip address` (또는 `ifconfig`): 가상머신 IP 확인
- `ping www... / DNS`: 네트워크 연결 확인
- **스냅샷**: 특정 시점의 상태를 저장해 되돌릴 수 있는 기능 (실습 전 백업용)

## 2. Linux 명령어 기초
프롬프트 구조:
```
[ root @ localhost ~ ] #
  └계정명┘  └호스트명┘ └홈디렉터리┘ └#=root, $=일반사용자┘
```

| 명령 | 설명 |
|---|---|
| `/` | 리눅스에서 가장 최상위 디렉터리(루트) |
| `pwd` | 현재 디렉터리 확인 (print working directory) |
| `cd <경로>` | 디렉터리 이동 (change directory) |
| `useradd <이름>` | 신규 유저 생성 |
| `su <이름>` | 사용자 변경 (switch user) |
| `mkdir <이름>` | 신규 디렉터리 생성 |
| `rmdir <이름>` | 디렉터리 삭제 (내용이 비어 있어야 함) |
| `rm -rf <이름>` | 디렉터리 강제 삭제 (내용까지) |
| `touch <이름>` | 빈 파일 생성 |

- **상대경로**: 현재 디렉터리 기준 / **절대경로**: 최상위(`/`)부터 전부 표기

### vi 에디터
```
vi <파일>     # 파일 열기/실행
```
| 키 | 동작 |
|---|---|
| `i` | 입력(Insert) 모드 진입 |
| `Esc` | 명령 모드로 복귀 |
| `:wq` | 저장하고 종료 |
| `:q!` | 저장하지 않고 강제 종료 |
| `:w` | 저장만 하기 |
| `:set nu` | 라인 번호 표시 |

## 3. 파일 권한 (Permission)
```
-  rwx  rwx  rwx
│  └owner┘└group┘└other┘
└종류(- 파일, d 디렉터리)
   r=read, w=write, x=execute
```
- 그룹 개념: `root` 그룹(root, root2…) / `users` 그룹(test1, test2…)

### 권한 숫자 (chmod)
| 권한 | 값 |
|---|---|
| read | 4 |
| write | 2 |
| execute | 1 |

```
chmod 755 test.txt
      │ │ └ other = 5 (r-x = 4+1)
      │ └ group = 5 (r-x)
      └ owner = 7 (rwx = 4+2+1)
```
- 7 = rwx, 6 = rw-, 5 = r-x, 4 = r-- …

## 4. 웹 서비스 구조와 APM
```
    Client(브라우저)
         │ 요청/응답
    Web 서버 ──┐
         │     │
    DB 서버(SQL)┘
```

### APM 구성 요소
| 약자 | 역할 | 예시 |
|---|---|---|
| **A** | Web Server | **Apache** (httpd) — webpage 제공 |
| **P** | 스크립트 언어 | **PHP** |
| **M** | DB Server | **MariaDB** |

### CSR vs SSR
- **CSR (Client-Side Rendering)**: 브라우저(클라이언트)에서 처리 — HTML
- **SSR (Server-Side Rendering)**: 서버에서 처리 후 결과 전달 — PHP, ASP, JSP
- 웹 서버는 CSR용 정적 파일은 그대로 보내고, SSR 코드는 서버에서 **실행한 결과(결과물)** 만 브라우저로 보낸다.

## 5. Apache(httpd) 웹 서버 설치
```
dnf install -y httpd php mariadb-server     # 패키지 관리 모듈(-y: 설치 중 전부 yes)
rpm -qa | grep <패키지>                      # 설치 여부 확인(파이프: 앞 결과를 뒤 명령으로)
```

### 방화벽 (firewall-cmd)
기본적으로 http 80 포트가 **닫혀 있다** → 열어줘야 외부 접속 가능.
```
firewall-cmd --permanent --add-port=80/tcp      # 영구 적용, 포트로 열기
firewall-cmd --permanent --add-service=http      # 또는 서비스명으로 열기
firewall-cmd --reload                            # 변경 사항 적용
```
- `--permanent` 없이 열면 **재부팅 시 사라진다**.

### httpd 모듈 방식
- **static modules**: 미리 컴파일되어 들어감 (적용 고정)
- **shared modules**: 필요 시 동적 로드 (변경·확장 용이)

### 첫 웹페이지 만들기
```
cd /var/www/html        # Apache 기본 문서 루트
vi index.html           # 시작 페이지 파일 (브라우저가 기본으로 찾는 파일)
```

## 6. 정정·보충
- `rm -rf`는 복구가 어려우므로 경로를 반드시 확인하고 사용한다 (특히 `rm -rf /` 류 주의).
- Promiscuous mode는 같은 세그먼트라도 스위치 환경에서는 포트 미러링(SPAN) 없이는 모든 패킷이 보이지 않을 수 있다.
- `chmod`는 숫자(8진수) 방식 외에 기호 방식(`chmod u+x`)도 있다.

## 7. 복습 질문
1. `chmod 640 파일`은 owner/group/other에게 각각 어떤 권한을 주는가?
2. 상대경로와 절대경로의 차이를 예로 들면?
3. APM의 A·P·M은 각각 무엇을 담당하는가?
4. 방화벽 규칙에 `--permanent`를 빼면 어떤 일이 생기나?
