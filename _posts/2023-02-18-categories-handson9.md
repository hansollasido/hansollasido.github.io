---
title: "[Chap 9]"
excerpt: "비지도 학습"

categories:
  - 핸즈온
tags:
  - [공부, 머신러닝]

use_math: true

permalink: /categories2/handson9/

toc: true
toc_sticky: true

date: 2023-02-18
last_modified_at: 2023-02-18
---

# 비지도 학습

레이블 없이 학습해야하는 **비지도 학습 알고리즘**이 필요함
- 거의 대부분의 데이터들은 레이블이 없기 때문임

## 군집
- 비슷한 것끼리 그룹을 짐
- 그러한 그룹을 클러스터(cluster)라고 함

<img src="../../assets/images/021801.png" width="500px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

왼쪽은 레이블이 있고, 오른쪽은 레이블이 없음

---
군집이 사용되는 애플리케이션

1. 고객 분류
2. 데이터 분석
3. 차원 축소 기법
4. 이상치 탐지
5. 준지도 학습
6. 검색 엔진
7. 이미지 분할

클러스터에 대한 보편적인 정의는 없고 상황에 따라 다름. 어떤 알고리즘은 **센트로이드**라 부르는 특정 포인트를 중심으로 모인 샘플을 찾음. 종류는 아주 많음.