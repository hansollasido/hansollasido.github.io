---
title: "Linux"
excerpt: "CPU, Cache 확인"

categories:
  - Linux
tags:
  - [Linux]

permalink: /categories17/linux/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-16
last_modified_at: 2023-08-16
---

**전체적인 컴퓨터 구조 및 용량 확인 가능**
```
lscpu
```

**cache 확인**
```
cat /sys/devices/system/cpu/cpu0/cache/index0/...
```

위 명령어로 cache 정보를 확인할 수 있음. 

**apt-get**
```
apt-get install (library 이름)
```

**컴퓨터 구조 확인**
```
uname -m
```