# 2026-09-14 · 클라우드 컴퓨팅 기초 (IaaS/PaaS/SaaS · Docker/K8s · MSA)

> 원본: [original/Notes_260914_102255_1.pdf](original/Notes_260914_102255_1.pdf)
>
> ℹ️ 이 필기의 **뒷부분(p3~)은 채용·면접 준비(인재상·STAR·MECE 등) 메모**다. 성격이 달라 해당 내용은 [`../_career/2026-09-14/notes.md`](../_career/2026-09-14/notes.md)에 따로 정리했다. (원본 PDF는 이 폴더에 보관)

## 1. 클라우드 컴퓨팅 기초
- Database → 근거화(구조화)하여 저장.
- 전통 네트워크 구성: `Server – 라우터 – 스위치 – PC…` → 최신은 **Server를 Cloud 형태로 구성**.

### 3-tier 구조
```
Front-End  ↔  Network  ↔  Back-End
```
- Back-End → 안정성이 중요(↑).
- **API**: 소프트웨어 간에 **정보 교환 및 상호작용을 위한 규약**.

## 2. 전통 인프라 vs 클라우드
- **온프레미스(전통적 인프라)**: 실제 물리적인 data server가 필요.
  - 단점: 하드웨어 장비·추가 설치 비용·시간 소요.
- **클라우드**: 사용한 만큼 과금(종량제) → 실제 사용량 변동에 유연하게 대응.
  - AI를 통해 "인프라 운영 + 데이터 활용" 문제를 해결하는 방향.

## 3. 배포 모델과 서비스 모델
- 배포 모델: **private – public – hybrid cloud**
- 서비스 모델 단계: **On-premise → IaaS → PaaS → SaaS** (위로 갈수록 관리 부담↓, 추상화↑)
  - 흐름상 SaaS → PaaS 방향으로도 활용(개발 유연성 확보).

| 모델 | 제공 범위 | 비유 |
|---|---|---|
| IaaS | 서버·스토리지·네트워크(인프라) | 빈 땅·건물 |
| PaaS | 개발·실행 플랫폼 | 주방 완비된 공간 |
| SaaS | 완성된 소프트웨어 | 완성된 요리 |

## 4. CSP / MSP
- **CSP (Cloud Service Platform)**: **AWS / Azure / GCP**.
- **MSP (Managed Service Provider)**: CSP와 Client를 연결해주는 역할(운영 대행).

### AWS 핵심 서비스 예
| 서비스 | 역할 |
|---|---|
| **EC2** | 가상 서버(컴퓨팅) |
| **S3** | 객체 스토리지 |
| **VPC** | 가상 사설 네트워크 |

## 5. 클라우드 서비스의 강점
- **확장성 / 고가용성**.

## 6. 컨테이너·오케스트레이션
- **Docker / Container**: `Virtualized deployment → Container deployment`로 진화(가볍고 빠른 배포).
- **Kubernetes Architecture**: 다수의 docker(컨테이너) 관리.
  - Kubernetes Cluster → 무중단 업데이트: **Rolling update**.
- **MSA (Micro Service Architecture)**: 서비스를 작은 단위로 분리.
  - **DevOps**: Development ↔ Operations — 개발과 운영을 동시에 진행하며 협업.
  - **Telemetry**: 운영 지표 수집·관측.

## 7. 정정·보충
- IaaS/PaaS/SaaS는 "관리 책임 경계"가 다르다 — SaaS로 갈수록 사용자가 관리할 영역이 줄어든다.
- Rolling update 외에도 Blue-Green, Canary 배포 방식이 있다(무중단 배포 전략).
- VPC는 클라우드 안에서 격리된 사설망 — 보안 그룹·서브넷으로 접근을 통제한다.

## 8. 복습 질문
1. IaaS·PaaS·SaaS의 관리 책임 경계는 어떻게 다른가?
2. 온프레미스 대비 클라우드의 과금 방식(종량제) 장점은?
3. Kubernetes의 Rolling update가 해결하는 문제는?
4. CSP와 MSP의 역할 차이는?
