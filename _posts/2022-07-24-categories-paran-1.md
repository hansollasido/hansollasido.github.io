---
title: "[📬] 파란학기제"
excerpt: "아주대 파란학기제 : 주차면 관제 AI 시스템 구축"

categories:
  - 파란학기제
tags:
  - [대회, 수상]

permalink: /categories3/paran/

toc: true
toc_sticky: true

date: 2023-01-02
last_modified_at: 2023-01-02
---

## 파란학기제

2023년도 2학기 파란학기제 기업연계 프로젝트를 진행

[베스텔라랩](https://vestellalab.com/)과 연계하여 "CCTV 기반 주차면 관제 인공지능 시스템 구축" 과제 수행

yolov5를 기반으로 한 object detection과 주차면 ROI의 IoU를 계산하고 IoU값이 특정값 이상이면 FULL로 아니면 EMPTY로 판단하게 설계

정확한 object detection을 하기 위해 전이학습을 진행

### 전체적인 알고리즘

1. 주차면 ROI와 Object Detection Box의 IoU를 계산

2. 몇 초 이상 특정값 이상이면 FULL로 판단

3. FULL이었을 때 몇 초 이상 특정값 이하이면 EMPTY로 판단

4. 지나가는 차량에 의해 가려졌을 때, Object Detection이 되지 않는 경우가 있음
  - IoU의 데이터를 분석하여 출입 판단을 통해 FULL을 유지할 수 있도록 설계

5. ROI와 IOU의 위치 관계식을 통한 FULL / EMPTY 판단 알고리즘 설계

**해당 알고리즘은 아래 영상으로 자세히 설명 가능**

---

### 중간교류회

<iframe width="853" height="480" src="https://www.youtube.com/embed/svPo7Vf9rx4" title="2022-2 중간교류회 꾸리스탈팀" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

### ppt 설명

- 주차면 관제 영상 시스템 구축 관련 기술적 내용 설명

<iframe width="853" height="480" src="https://www.youtube.com/embed/jDdJ-bBVO1A" title="꾸리스탈 PT 영상" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

### 성과발표회

<iframe width="853" height="480" src="https://www.youtube.com/embed/MMBQsY_0hvc" title="2022 2 파란학기 성과발표회" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

---

### 아주대학교 총장상 (훌륭한 뱃사공상(일등상)) 수상

<img src="../../assets/images/010601.jpg" width="800px" height="800px" title="수상" alt="수상"><img><br/>
