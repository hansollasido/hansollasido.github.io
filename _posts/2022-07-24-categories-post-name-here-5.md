---
title: "[👨‍🔧] 아주대 학습 공동체 스터디 (opencv)"
excerpt: "OpenCV 4로 배우는 컴퓨터 비전과 머신러닝"

categories:
  - 학습공동체
tags:
  - [C++, opencv]

permalink: /categories5/post-name-here-5/

toc: true
toc_sticky: true

date: 2022-10-31
last_modified_at: 2022-10-31
---

## 학습공동체

OpenCV 4로 배우는 컴퓨터 비전과 머신러닝 책으로 스터디 진행

언어는 C++

<img src="../../assets/images/102807.jpg" width="450px" height="300px" title="OP code 예시" alt="OP code"><img><br/>

## 주요 함수 설명

```cpp

Mat imread(const String& filename, int flags = IMREAD_COLOR);

```

- filename : 불러올 영상 파일 이름
- flags : 영상 파일 불러오기 옵션 플래그. ImreadModes 열거형 상수를 지정
- 반환값 : 불러온 영상 데이터(Mat 객체)

잘못된 형식의 영상 파일이라면 비어 있는 Mat 객체를 반환하기 때문에, imread()함수 사용 뒤에 empty()를 사용해서 제대로 생성되었는지 확인하는 것이 좋음

`RGB가 아닌 BGR로 불러온다는 것을 확인!`

저장하기 위해서는 imwrite()를 사용

```cpp
bool imwrite(const String& filename, InputArray img, const std::vector<int>& params = std::vector<int>());
```

- filename : 저장할 영상 파일 이름
- img : 저장할 영상 데이터(Mat 객체)
- params : 저자알 영상 파일 형식에 의존적인 파라미터(플래그 & 값) 쌍
- 반환값 : 정상적으로 저장하면 true, 실패하면 false를 반환함

params 인자의 형식은 std::vector<int> 타입으로 지정하며, 옵션 플래그와 실제 값을 정수 값 두 개의 쌍으로 지정해야함

JPEG 압축률을 95%로 지정하고 싶다면

```cpp
vector<int> params;
params.push_back(IMWRITE_JPEG_QUALITY);
params.push_back(95);
imwrite("lenna.jpg",img,params);
```

영상 데이터가 잘 준비되었는지 확인

```cpp
bool Mat::empty() const
```
- 반환값 : 행렬의 rows 또는 cols 멤버 변수가 0이거나, 또는 data 멤버 변수가 NULL이면 true를 반환함

빈 창 생성

```cpp
void namedWindow(const String& winname, int flags = WINDOW_AUTOSIZE);
```
- winname : 영상 출력 창 상단에 출력되는 창 고유 이름. 이 문자열로 창을 구분함
- flags : 생성되는 창의 속성을 지정하는 플래그. WindowFlags 열거형 상수를 지정

고유한 문자열을 부여하여 각각의 창을 구분함

namedWindow() 함수의 flags 인자 기본값은 WINDOW_AUTOSIZE임

destroyWindow() 또는 destroyAllWindows() 함수를 이용해서 창을 닫음

```cpp
void destroyWindow(const String& winname);
void destoryAllWindows();
```

- winname 소멸시킬 창 이름

창 위치 바꿀 때 moveWindow() 사용

```cpp
void moveWindow(const String& winname, int x, int y);
```

- winname : 위치를 이동할 창 이름
- x : 창이 이동할 위치의 x 좌표
- y : 창이 이동할 위치의 y 좌표

출력 창의 크기를 변경하고 싶다면 resizeWindow() 함수 사용

```cpp
void resizeWindow(const String& winname, int width, int height);
```

- winname : 크기를 변경할 창 이름
- width : 창의 가로 크기
- height : 창의 세로 크기

이 때 width와 height는 창 전체 크기가 아니라 창의 뷰 영역에 나타나는 영상 크기를 의미함

WINDOW_AUTOSIZE 플래그를 사용하여 만들어진 영상 출력 창은 resizeWindow() 함수로 크기를 변경할 수 없음

추력해주는 inshow() 함수를 살펴보겠음

```cpp
void imshow(const String& winname, InputArray mat);
```

- winname : 영상을 출력할 대상 창 이름
- mat : 출력할 영상 데이터(Mat 객체)

mat 객체에 저장된 영상이 1채널 8비트 uchar 자료형으로 구성된 grayscale 영상이라면 픽셀 값을 그대로 그레이 스케일 밝기 형태로 나타냄

mat 객체에 저장된 영상이 uchar 자료형을 사용하는 3채널 컬러 영상이라면 색상 채널이 BGR 순서로 되어 있다고 간주하여 색상 표현

만약 mat 객체가 부호 없는 16비트 또는 32비트 정수형이라면 행렬 원소 값을 256으로 나눈 값을 영상 밝기 값으로 사용함

반면 mat 객체가 32 비트 또는 64 비트 실수형 행렬이라면 행렬 원소에 255를 곱한 값을 밝기 값으로 사용함

사용자로부터 키보드 입력을 받는 용도로 사용되는 함수 waitKey() 함수

```cpp
int waitKey(int delay = 0);
```

- delay : 키 입력을 기다릴 시간(밀리초 단위). delay <= 0이면 무한히 기다림
- 반환값 : 눌린 키 값. 지정한 시간 동안 키가 눌리지 않으면 -1을 반환함

imshow만으로는 영상이 화면에 나오지 않음

