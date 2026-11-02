# 2026-08-28 · MariaDB 권한 관리 (GRANT / REVOKE)

> 원본: [original/0828_260828_103029_1.pdf](original/0828_260828_103029_1.pdf)

짧은 세션. 08-27의 DB 내용에 이어 **사용자 권한 부여/회수**를 정리한다.

## 1. 권한 부여 — GRANT
```sql
GRANT ALL PRIVILEGES ON test.* TO 'hong'@'%';   -- test DB의 모든 테이블 권한을 hong에게
```
- `test.*` → `데이터베이스.테이블` (`*`는 전체).
- `'hong'@'%'` → 사용자 `hong`, `@` 뒤는 접속 허용 호스트. `%` = **모든 호스트에서 접속 허용**(`localhost`면 로컬만).
- `ALL PRIVILEGES` 대신 `SELECT, INSERT, UPDATE` 처럼 **필요한 권한만** 주는 것이 최소 권한 원칙.

## 2. 권한 회수 — REVOKE
```sql
REVOKE ALL PRIVILEGES ON test.* FROM 'hong'@'%';   -- 부여했던 권한 회수
```
- GRANT는 `TO`, REVOKE는 `FROM`을 쓴다.

## 3. 보충
- 권한 변경 후에는 `FLUSH PRIVILEGES;`로 즉시 반영(계정 생성·권한표 직접 수정 시 특히).
- 현재 권한 확인: `SHOW GRANTS FOR 'hong'@'%';`
- `'hong'@'%'`와 `'hong'@'localhost'`는 **서로 다른 계정**으로 취급된다 → 회수할 때 호스트까지 정확히 맞춰야 한다.
- `%`로 모든 호스트를 여는 것은 편하지만 보안상 위험 → 접속이 필요한 대역/호스트로 제한하는 것이 바람직.

## 4. 복습 질문
1. `'user'@'%'`와 `'user'@'localhost'`의 차이는?
2. GRANT와 REVOKE에서 각각 쓰는 전치사(TO/FROM)는?
3. 최소 권한 원칙에 따르면 웹앱 전용 DB 계정에 어떤 권한만 주는 게 좋을까?
