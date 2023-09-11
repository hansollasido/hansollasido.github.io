---
title: "Docker"
excerpt: "Docker 실행"

categories:
  - Docker
tags:
  - [Docker]

permalink: /categories18/docker/

use_math: true
toc: true
toc_sticky: true

date: 2023-08-16
last_modified_at: 2023-08-16
---

## Docker

맨 처음에, 다음과 같이 생성할 수 있음. 

```
docker ps

docker image ls

docker run -it --privileged --name hansol_gem5 gem5:fs_230706 /bin/bash
```

만일 docker가 실행되지 못하면 attach하거나 run해줘야 함.

```
docker attach 
```

### 기본적인 설명

docker은 가상 환경으로 container를 만들어 HW를 효율적으로 분배하여 사용 가능함. 

어떻게 보면 VM과 비슷하나 차이점이 있음.

<p align="center"><img src="../../assets/images/081602.jpg" width="500px" height="500px" title="Docker" alt="Docker" ><img></p>

VM은 Host OS 위에 Guest OS를 올리는 형식.

Docker은 Guest OS 없이 Host OS 위에 사용하고 싶은 Bins/Libs를 올리는 형식.

즉 VM은 Kernel이 각 사용자마다 공유되지 않지만, Docker은 공유가 가능.

공유된 Kernel을 사용하므로 Docker은 비교적 가볍고 빠름. 

하지만 단점도 존재함. 보안성이 취약해서 다른 사용자가 쉽게 제어할 수 있음. 보안 측면에서는 VM이 더 좋음. 

