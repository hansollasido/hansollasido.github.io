---
title: "[💪] 아주대 학습 공동체 스터디 (opencv) 3"
excerpt: "Mat 클래스"

categories:
  - 학습공동체
tags:
  - [C++, opencv]

permalink: /categories6/post-name-here-9/

toc: true
toc_sticky: true

date: 2022-10-31
last_modified_at: 2022-10-31
---

## Mat 클래스

- opencv에서 가장 많이 사용하는 클래스
- 행렬의 복사, 부분 행렬 추출, 행렬의 정보 참조 방법 등을 설명하고 있음

### Mat 클래스 개요

Mat 클래스는 2차원 행렬뿐만 아니라 고차원 행렬을 표현할 수 있음

한 개 이상의 채널을 가질 수 있음

정수, 실수, 복소수 등으로 구성된 행렬 또는 벡터를 저장할 수 있음

경우에 따라 벡터 필드, 포인트 크라우드, 텐서, 히스토그램 등 정보를 저장하는 용도로 쓰임

Mat 클래스의 멤버 변수
- Mat::dims는 행렬의 차원, 영상과 같은 2차원 행렬의 경우 2
- Mat:rows와 Mat::cols 멤버 변수는 2차원 행렬의 크기, 3차원일 경우 둘다 -1을 저장함
- Mat::size로 3차원 이상 행렬의 크기 참조 가능
- Mat::data 행렬의 원소 데이터가 저장되어 있는 메모리 공간을 가리키는 포인터형 멤버 변수
- 아무것도 저장되어 있지 않은 상태면 Mat::data는 0(NULL)임

다양한 자료형을 지원함

행렬이 어떤 자료형을 사용하는지에 대한 정보를 깊이(depth)라고 부름

CV_<bit-depth>{U|S|F}

채널도 표현 가능함

CV_<bit-depth>{U|S|F}C(number_of_channels)

### 행렬의 생성과 초기화

```cpp
Mat img1;
```

이렇게 생성된 img1 객체는 비어 있는 행렬임

Mat 객체를 생성함에 동시에 원소 값 저장을 위한 메모리 공간을 할당하려면 다음 생성자를 사용해야함

```cpp
Mat::Mat(int rows, int cols, int type);
```

- rows : 새로 만들 행렬의 행 개수
- cols : 새로 만들 행렬의 열 개수
- type : 새로 만들 행렬의 타입

```cpp
Mat img2(480,640,CV_8UC1); // unsigned char, 1-channel
Mat img3(480,640,CV_8UC3); // unsigned char, 3-channels
```

크기 지정할 때 Size 클래스 사용 가능

```cpp
Mat::Mat(Size size, int type);
```
- size : 새로 만들 행렬의 크기, Size(cols, rows) 또는 Size(width, height)
- type : 새로 만들 행렬의 타입

```cpp
Mat img4(Size(640, 480), CV_8UC3); // Size(width, height)
```

쓰레기 값으로 채워지기 때문에 웬만해서는 초기화 하는 것이 좋음

```cpp
Mat::Mat(int rows, int cols, int type, const Scalar& s);
Mat::Mat(Size size, int type, const Scalar& s);
```

- rows : 새로 만들 행렬의 행 개수
- cols : 새로 만들 행렬의 열 개수
- size : 새로 만들 행렬의 크기
- type : 새로 만들 행렬의 타입
- s : 행렬 원소 초깃값

s를 설정하여 초기값 설정

예를 들어 빨간색으로 설정된 칼라 영상을 생성하려면

```cpp
Mat img5(480, 640, CV_8UC1, Scalar(128)); // initial values, 128
Mat img6(480, 640, CV_8UC3, Scalar(0,0,255)); // initial values, red
```

모든 원소를 으로 초기화된 행렬을 만드는 함수는 Mat::zeros()임

```cpp
static MatExpr Mat::zeros(int rows, int cols, int type);
static MatExpr Mat::zeors(Size size, int type);
```
- 반환값 : 모든 원소가 0으로 초기화된 행렬 표기식

예를 들어 0으로 초기화된 3x3 정수형 행렬을 생성하려면

```cpp
Mat mat1 = Mat::zeros(3, 3, CV_32SC1); //0's matrix
```

ones나 eyes도 사용가능

```cpp
static MatExpr Mat::ones(int rows, int cols, int type);
static MatExpr Mat::ones(Size size, int type);

static MatExpr Mat::eye(int rows, int cols, int type);
static MatExpr Mat::eye(Size size, int type);
```

예제

```cpp
Mat mat2 = Mat::ones(3, 3, CV_32FC1); // 1's matrix
Mat mat3 = Mat::eye(3, 3, CV_32FC1); // identity matrix
```

행렬 원소를 저장할 메몰 공간을 새로 할당하는 것이 아니라 기존에 이미 할당되어 있는 메모리 공간의 데이터를 행렬 원소 값으로 사용할 수 있음

외부 메모리 공간을 활용하여 Mat 객체를 생성한다는 것은 자체적인 메모리 할당을 수행하지 않고 외부 메모리를 참조하는 방식이기 때문에 객체 생성이 빠르다는 장점이 있음

```cpp
Mat::Mat(int rows, int cols, int type, void* data, size_t step=AUTO_STEP);
Mat::Mat(Size size, int type, void* data, size_t step=AUTO_STEP);
```

- data : 사용할 (외부) 행렬 데이터의 주소, 외부 데이터를 사용하여 Mat 객체를 생성할 경우, 생성자에서 원소 데이터 저장을 위한 메모리 공간을 동적으로 할당하지 않음


여섯 개의 원소를 갖는 float 자료형의 배열 data를 먼저 정의하고, 이 배열을 행렬 원소로 사용하는 Mat 객체 mat4를 생성함

```cpp
float data[] = {1, 2, 3, 4, 5, 6};
Mat mat4(2, 3, CV_32FC1, data);
```

- Mat 객체의 원소 값과 외부 데이터 공간의 데이터 값이 상호 공유된다는 점을 기억해야함
- 동적 할당 가능하나 자동으로 해제되지 않음

```cpp
Mat_<float> mat5_(2,3);
mat5_ << 1, 2, 3, 4, 5, 6;
Mat mat5 = mat5_;
```

간략하게 아래와 같이 쓸 수 있음

```cpp
Mat mat5 = (Mat_<float>(2, 3) << 1, 2, 3, 4, 5, 6);
```

