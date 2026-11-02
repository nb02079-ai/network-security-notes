# 2026-09-17 · Random Subspace Method · 앙상블 분류기 (논문 스터디)

> 원본: [original/Notes_260917_092207_1.pdf](original/Notes_260917_092207_1.pdf)
>
> 필기 상단에 "Shell script"라고 적혀 있으나 실제 내용은 머신러닝 앙상블(Random Subspace) 스터디입니다.

## 1. 데이터와 ML 구분
- 여러 데이터를 통합해서 ML, 각 데이터로 학습된 ML → **Assemble(앙상블)** (차이).
- 개념: **성능 + 모델 간 상호(상관)관계 + 다양성 관계 설명**.

## 2. Random Subspace Method
- database → 여러 집단(부분공간)으로 나누어 학습.
  - tree가 생성될 때 → 특징(feature) 공간/경로 분류의 결정들이 어떻게 연결되는가.
- 효과: **개별 성능↑ + 모델 간 상관성↓** → 여러 classifier가 서로 다른(같은) feature를 공유 가능.
- 핵심: 서로 다른 **feature 부분집합(subspace)** 으로 학습해 분류기들의 **다양성**을 확보.

## 3. Classification by Ensembles from Random Partitions
- 서로 겹치지 않는(mutually exclusive) predictor(변수)의 집합으로 분할.
- 위한 경험: **subspace 크기 / feature 개수가 중요**.
- (필기 약어) `C-T CERP` / `LR-T CERP` — CERP(Classification by Ensembles from Random Partitions)의 변형(Classification Tree 기반 / Logistic Regression Tree 기반 등).

## 4. 메모 성격
앙상블 학습(Random Subspace, Random Partition 기반 CERP)의 **핵심 아이디어 정리** — 개별 분류기 성능과 분류기 간 다양성(상관성 감소)의 균형이 앙상블 성능을 좌우한다는 내용. 논문/과제용 스터디 노트로 보임.

## 5. 복습 질문
1. 앙상블에서 "개별 성능"과 "다양성"이 함께 필요한 이유는?
2. Random Subspace Method가 다양성을 만드는 방식은?
3. CERP가 Random Subspace와 다른 분할 관점은 무엇인가?
