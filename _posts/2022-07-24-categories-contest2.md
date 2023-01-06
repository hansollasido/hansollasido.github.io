---
title: "[국민대 자율 주행 자동차 대회]"
excerpt: "스케일카 기반 국민대 자율 주행 자동차 대회 우수상 수상"

categories:
  - 자율주행
tags:
  - [대회, 수상]

permalink: /categories4/contest2/

toc: true
toc_sticky: true

date: 2022-07-24
last_modified_at: 2022-07-24
---

## 국민대 자율주행자동차 대회 (우수상)

<img src="../../assets/images/010603.jpg" width="800px" height="800px" title="수상" alt="수상"><img><br/>

OpenCV를 이용하여 Line Detection 역할 담당

흰색 line을 쫓아가는 것을 기본으로 하여 라바콘, 동적 장애물, 정적 장애물, slow 구간에 맞춰 빠른 시간내에 트랙을 완주하는 것이 목표

<img src="../../assets/images/010602.jpg" width="800px" height="800px" title="수상" alt="수상"><img><br/>

### line detection

<iframe width="700" height="400" src="../../assets/images/010604.mp4" title="2022 2 파란학기 성과발표회" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

- camera를 bird eye 버젼으로 변경

- 이미지 전처리를 활용하여 흰색 차선을 detection한 뒤, 트랙 중앙의 위치를 가져옴

- 위치 정보를 활용하여 차량 제어함