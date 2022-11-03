---
title: "[💪] 아주대 학습 공동체 스터디 (opencv) 3"
excerpt: "Mat 클래스"

categories:
  - 학습공동체
tags:
  - [C++, opencv]

permalink: /categories6/study-group-2/

toc: true
toc_sticky: true

date: 2022-10-31
last_modified_at: 2022-11-01
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

리스트로 초기화 하는 방법도 있음

```cpp
Mat mat6 = Mat_<float>({2, 3}, {1, 2, 3, 4, 5, 6});
```

비어 있는 Mat 객체 또는 이미 생성된 Mat 객체에 새로운 행렬을 할당하려면 Mat 클래스의 Mat::create() 멤버 함수를 사용할 수 있음

```cpp
void Mat::create(int rows, int cols, int type);
void Mat::create(Size size, int type);
```

이미 행렬 데이터가 할당되어 있는 Mat 객체에서 함수를 호출할 경우, 만약 Mat::create() 함수의 인자로 지정한 행렬 크기와 타입이 기존 행렬과 모두 같으면 Mat::create()함수는 종료

하지만 다를 경우 Mat::create() 함수는 일단 기존 메모리 공간을 해제한 후, 새로운 행렬 데이터 저장을 위한 메모리 공간을 할당함

예) 이미 생성되어 있는 Mat 클래스 타입의 변수 mat4와 mat5에 새로운 크기와 타입의 행렬을 할당하려면 다음과 같이 코드를 작성해야함

```cpp
mat4.create(256, 256, CV_8UC3); // 256x256, uchar, 3-channels
mat5.create(4, 4, CV_32FC1);
```

원소 값을 초기화하는 기능이 없음

`행렬 전체 원소 값을 초기화하고 싶으면 Mat::setTo()로 설정함`

```cpp
Mat& Mat::operator = (const Scalar& s);
```

- s : 행렬 원소의 설정할 값
- 반환값 : 값이 설정된 Mat 객체의 참조

```cpp
Mat& Mat::setTo(InputArray value, InputArray mask = noArray());
```

- value : 행렬 원소에 설정할 값
- mask : 마스크 행렬, 마시크 행렬의 원소가 0이 아닌 위치에서만 value 값이 설정됨. 행렬 전체 원소 값을 설정하려면 noArray() 또는 Mat()을 지정함
- 반환값 : Mat 객체의 참조

Mat::setTo() 함수는 두 개의 인자를 가지고 있지만 두 번째 인자 mask는 기본값을 가지고 있으므로 생략 가능

```cpp
mat4 = Scalar(255, 0, 0);
mat5.setTo(1.f);
```

create하고 setTo해서 초기화 가능 

```cpp
void MatOp1()
{
  Mat img1; // empty matrix

  Mat img2(480, 640, CV_8UC1); // unsigned char, 1-channel
  Mat img3(480, 640, CV_8UC3); // unsigned char, 3-channel
  Mat img4(Size(640, 480), CV_8UC3); // Size(width, height)

  Mat img5(480, 640, CV_8UC1, Scalar(128)); // initial values, 128
  Mat img6(480, 640, CV_8UC3, Scalar(0, 0, 255)) // initial values, red

  Mat mat1 = Mat::zeros(3, 3, CV_32SC1); // 0's matrix
  Mat mat2 = Mat::ones(3, 3, CV_32FC1); // 1's matrix
  Mat mat3 = Mat::eye(3, 3, CV_32FC1); // identity matrix

  float data[] {1, 2, 3, 4, 5, 6};
  Mat mat4(2, 3, CV_32FC1, data);

  Mat mat5 = (Mat_<float>(2, 3) << 1, 2, 3, 4, 5, 6);
  Mat mat6 = Mat_<uchar>({2, 3}, { 1, 2, 3, 4, 5, 6});
  
  mat4.create(256, 256, CV_8UC3); // uchar, 3-channels
  mat5.create(4, 4, CV_32FC1); // float, 1-channel

  mat4 - Scalar(255, 0, 0);
  mat5.setTo(1.f);
}
```

### 행렬의 복사

강아지 사진이 담겨 있는 dog.bmp 파일을 불러와서 Mat type 변수 img1에 저장하고 이를 이용하여 다양한 예제 코드를 만들어 보겠음

```cpp
Mat img1 = imread('dog.bmp');
```

복사하는 데에 가장 간단한 방법은 대입 연산자를 사용하는 방법임

```cpp
Mat img2 = img1; // 복사 생성자(얕은 복사)
```

img1과 같은 크기, 같은 타입의 새로운 Mat 객체 img2를 생성하고 참조하도록 설정함

즉, img1과 img2는 하나의 영상을 공유하는 서로 다른 이름의 변수 형태로 동작함

이를 **얕은 복사(shallow copy)**라 함

```cpp
Mat img3;
img3 = img1; // 대입 연산자(얕은 복사)
```

위와 같이도 가능함

```cpp
Mat Mat::clone() const;
```
- 반환값 : *this 행렬의 복사본

```cpp
void Mat::copyTo(OutputArray m) const;
void Mat::copyTo(OutputArray m, InputArray mask) const;
```
- m : 복사본이 저장될 행렬, 만약 *this 행렬과 크기 및 타입이 다르면 메모리를 새로 할당한 후 픽셀 값을 복사함
- mask : 마스크 행렬, 마스크 행렬의 원소 값이 0이 아닌 좌표에서만 행렬 원소를 복사함

Mat::clone() 함수는 완전히 Mat 객체를 새로 만들어서 반환함

Mat::copyTo() 함수는 인자로 전달된 m 행렬에 자기 자신을 복사함


```cpp
Mat img4 = img1.clone(); // 깊은 복사

Mat img5;
img1.copyTo(img5); // 깊은 복사
```

이와 같이 완전히 메모리 공간을 새로 할당하여 픽셀 값을 복사하는 형태의 복사를 깊은 복사(deep copy)라고 함

```cpp
#include "opencv2/opencv.hpp"
#include <iostream>

using namespace cv; //using namespace로 cv::를 안 적어도 됨
using namespace std; //std::를 안적어도 됨

void MatOp2()
{
	Mat img1 = imread("dog.bmp");

	Mat img2 = img1;
	Mat img3;
	img3 = img1;

	Mat img4 = img1.clone();
	Mat img5;
	img1.copyTo(img5);

	img1.setTo(Scalar(0, 255, 255)); //yellow
	imshow("img1", img1);
	imshow("img2", img2);
	imshow("img3", img3);
	imshow("img4", img4);
	imshow("img5", img5);

	waitKey();
	destroyAllWindows();
}



int main()
{
	MatOp2();
	//cout << "Hello OpenCV " << CV_VERSION << endl;
	//Mat img; 

	//img = imread("lenna.bmp");

	//if (img.empty()) {
	//	cerr << "Image load failed!" << endl;
	//	return -1;
	//}
	////namedWindow("image"); 
	////imshow("image", img);

	////waitKey();
	return 0;
}
```
<img src="../../assets/images/110101.png" width="450px" height="300px" title="결과" alt="OP code"><img><br/>

깊은 복사를 수행한 영상은 강아지 그대로 간직하고 있음

### 부분 행렬 추출

사각형 모양의 부분 영상을 추출하거나 참조하는 방법

Mat 클래스 괄호 연산자 재정의 함수를 사용

```cpp
Mat Mat::operator()(const Rect& roi) const;
Mat Mat::operator()(Range rowRange, Range colRange) const;
```

- roi : 사각형 관심 영역
- rowRange : 관심 행 범위
- colRange : 관심 열 범위
- 반환값 : 추출한 부분 행렬 또는 영상. 부분 영상의 픽셀 데이터를 서로 공유함

```cpp
Mat img1 = imread('cat.bmp');
Mat img2 = imread(Rect(220, 120, 340, 240));
```

즉 영상의 (220, 120) 좌표부터 340 x 240 크기만큼의 사각형 영상을 추출함

**이 때 깊은 복사가 아니라 얕은 복사라는 점을 알아야 함**

반전 시키는 예제

```cpp
img2 = ~img2;
```

img2를 반전시키면 img1도 반전되는 것을 볼 수 있음

Mat 클래스의 부분 영상 참조 기능은 입력 영상에 사각형 모양의 관심 영역(ROI, Region Of Interest)을 설정하는 용도로 사용할 수 있음

만일 독립된 메모리 영역을 확보하여 부분 영상을 추출하고자 한다면 Mat::clone() 함수를 함께 사용

```cpp
Mat img3 = img1(Rect(220, 120, 340, 240)).clone();
```

img1 영상과 img3 영상은 서로 다른 메모리 공간을 사용함 -> 따라서 img3이 변한다고 img1이 변하는 것은 아님

```cpp
void MatOp3()
{
  Mat img1 = imread("cat.bmp");

  if(img1.empty()){
    cerr << "Image load failed!" << endl;
    return;
  }

  Mat img2 = img1(Rect(220, 120, 340, 240));
  Mat img3 = img1(Rect(220, 120, 340, 240)).clone();

  img2 = ~img2;

  imshow("img1", img1);
  imshow("img2", img2);
  imshow("img3", img3);

  waitKey();
  destroyAllWindows();
}
```
<img src="../../assets/images/110102.png" width="450px" height="300px" title="결과" alt="OP code"><img><br/>

Rect 뿐만 아니라 rowRange, colRange, row, col 함수를 이용하여 부분 추출 가능

### 행렬의 원소 값 참조

**Mat::at() 함수 사용 방법**

opencv에서 제공하는 가장 직관적인 행렬 원소 접근 방법임

```cpp
template<typename _Tp> _Tp& Mat::at(int y, int x);
```

- y : 참조할 행 번호
- x : 참조할 열 번호
- 반환값 : (_Tp& 타입으로 형 변환된) y 번째 행, x번째 열의 원소 값(참조)

```cpp
Mat mat1 = Mat::zeros(3, 4, CV_8UC1);

for (int j = 0; j < mat1.rows; j++){
  for (int i = 0; i < mat1.cols; i++){
    mat1.at<uchar>(j, i)++;
  }
}
```

위는 행렬의 모든 원소 값을 1만큼 증가시키는 예제 코드

**Mat::ptr() 함수 사용 방법**

Mat 행렬에서 특정 행의 첫 번째 원소 주소를 반환함

```cpp
template<typename _Tp>
_Tp* Mat::ptr(int y)
```

- y : 참조할 행 번호
- 반환값 : (_Tp* 타입으로 형 변환된) y번째 행의 시작 주소

포인터를 이용하여 지정한 행의 원소에 접근함

```cpp
for (int j = 0; j < mat1.rows; j++) {
  uchar* p = mat1.ptr<uchar>(j);
  for (int i = 0; i < mat1.cols; i++) {
    p[i]++;
  }
}
```

Mat::at() 함수보다 빠르게 동작함, 임의 좌표 원소에 빈번하게 접근해야 하는 경우라면 Mat::at() 함수가 더 좋음

**MatIterator_ 반복자 사용 방법**

- 함수 인자로 전달된 값이 행렬의 크기를 벗어나면 에러가 발생하는데 이러한 단점을 해결하고자 반복자 개념을 도입하여 원소를 참조할 수 있는 방법 제공함

Mat::begin() 함수를 이용하여 첫 번째 위치를 얻고 Mat::end() 함수를 이용하여 마지막 원소 바로 다음 위치를 얻을 수 있음

```cpp
for (MatIterator_<uchar> it = mat1.begin<uchar>(); it != mat1.end<uchar>(); ++it){
  (*it)++;
}
```

안전하게 행렬의 모든 원소를 접근 할 수 있음

하지만 Mat::ptr() 사용 방법보다는 느리고 Mat::at() 함수처럼 임의의 위치에 자유롭게 접근 할 수 없어 잘 사용하지 않음

```cpp
void MatOp4()
{
	Mat mat1 = Mat::zeros(3, 4, CV_8UC1);
	
	// Mat::at() 함수로 접근
	for (int j = 0; j < mat1.rows; j++) {
		for (int i = 0; i < mat1.cols; i++) {
			mat1.at<uchar>(j, i)++;
		}
	}
	
	//
	for (int j = 0; j < mat1.rows; j++) {
		uchar* p = mat1.ptr<uchar>(j);
		for (int i = 0; i < mat1.cols; i++) {
			p[i]++;
		}
	}

	for (MatIterator_<uchar> it = mat1.begin<uchar>(); it != mat1.end<uchar>(); ++it) {
		(*it)++;
	}

	cout << "mat1:\n" << mat1 << endl;
}
```
<img src="../../assets/images/110103.png" width="450px" height="300px" title="결과" alt="OP code"><img><br/>

모든 원소가 3으로 끝나는 것을 볼 수 있음

### 행렬 정보 참조하기

```cpp
Mat img1 = imread("lenna.bmp");

cout << "Width: " << img1.cols << endl
cout << "Height: " << img1.rows << endl
```

영상크기가 출력됨

현재 다루고 있는 영상 객체가 3채널 컬러 영상인지를 확인하려면 Mat::type() 멤버 함수를 이용

그레이스케일은 CV_8UC1, 3채널 영상은 CV_8UC3

```cpp
if (img1.type() == CV_8UC1)
	cout << "img1 is a grayscale image." << endl
else if (img1.type() == CV_8UC3)
	cout << "img1 is a truecolor image." << endl
```

`img1 is a truecolor image`

type()로 확인할 수 있음

```cpp
static inline
std::ostream& operator << (std::ostream& ouot, const Mat& mtx)
```

- out : C++ 표준 출력 스트림 객체
- mtx : 출력할 행렬
- 반환값 : C++ 표준 출력 스트림 객체의 참조

```cpp
void MatOp5()
{
	Mat img1 = imread("lenna.bmp");

	cout << "Width: " << img1.cols << endl;
	cout << "Height: " << img1.rows << endl;
	cout << "Channels: " << img1.channels() << endl;

	if (img1.type() == CV_8UC1)
		cout << "img5 is a grayscale image." << endl;
	else if (img1.type() == CV_8UC3)
		cout << "img5 is a truecolor image." << endl;

	float data[] = { 2.f, 1.414f, 3.f, 1.732f };
	Mat mat1(2, 2, CV_32FC1, data);
	cout << "mat1:\n" << mat1 << endl;
}
```

<img src="../../assets/images/110307.png" width="200px" height="3200px" title="결과" alt="OP code"><img><br/>


### 행렬 연산

행렬 연산, 역행렬, 전치 행렬 등을 구할 수 있음

사칙 연산 가능

행렬 곱은 아래 코드와 같음

```cpp
MatExpr Mat::mul(InputArray m, double scale = 1) const;
```

- m : *this 행렬과 크기 및 타입이 같은 행렬, 또는 행렬 표현식
- scale : 추가적으로 곱할 값
- 반환값 : 두 행렬의 같은 위치 원소끼리 곱셈한 결과 행렬 표현식

```cpp
MatExpr Mat::inv(int method=DECOMP_LU) const;
```

- method : 역행렬 계산 방법. DECOMP_LU, DECOMP_SVD, DECOMP_EIG, DECOMP_CHOLESKY 중 하나를 지정할 수 있음
- 반환값 : 역행렬에 대한 행렬 표현식

전치행렬도 구할 수 있음
```cpp
MatExpr Mat::t() const;
```

- 반환값 : 전치 행렬에 대한 행렬 표현식

예제 코드
```cpp
void MatOp6()
{
	float data[] = { 1, 1, 2, 3 };
	Mat mat1(2, 2, CV_32FC1, data);
	cout << "mat1:\n" << mat1 << endl;
	
	Mat mat2 = mat1.inv();
	cout << "mat2:\n" << mat2 << endl;

	cout << "mat1.t():\n" << mat1.t() << endl;
	cout << "mat1 + 3:\n" << mat1 + 3 << endl;
	cout << "mat1 + mat2:\n" << mat1 + mat2 << endl;
	cout << "mat1 * mat2:\n" << mat1 * mat2 << endl;
}
```

<img src="../../assets/images/110308.png" width="200px" height="3200px" title="결과" alt="OP code"><img><br/>

### 크기 및 타입 변환 함수

행렬 타입을 변경할 때 Mat::convertTo() 함수 사용
```cpp
void Mat::convertTo( OutputArray m, int rtype, double alpha=1, double beta=0) const;
```

- m : 출력 행렬. 만약 m 행렬이 적절한 크기와 타입이 아닌 경우 행렬 원소 데이터를 새로 할당
- rtype : 원하는 출력 행렬의 타입, 만약 rtype이 음수이면 출력 행렬은 입력 행렬과 같은 타입을 가짐
- alpha : 추가적으로 곱할 값
- beta : 추가적으로 더할 값

주어진 행렬의 크기 또는 채널 수를 변경하는 함수인 Mat::reshape() 함수
```cpp
Mat Mat::reshape(int cn, int rows=0) const;
```

- cn : 새로운 채널 수, 만약 이 값이 0이면 채널 수를 변경하지 않음
- rows : 새로운 행의 수, 만약 이 값이 0이면 행의 개수를 변경하지 않음
- 반환값 : 모양이 변경된 행렬을 반환함

행렬의 모양을 변경시키지 않고 크기를 변경시키고 싶다면 Mat::resize() 함수를 사용

```cpp
void Mat::resize(size_t sz);
void Mat::resize(size_t sz, const Scalar& s);
```

- sz : 새로운 행 개수
- s : 새로 추가되는 행 원소의 초기값

이미 존재하는 행렬에 원소 데이터를 추가하고 싶을 때에는 Mat::push_back() 함수 사용

```cpp
template<typename _Tp> void Mat::push_back(const _Tp& elem);
template<typename _Tp> void Mat::push_back(const Mat_<_Tp>& elem);
template<typename _Tp> void Mat::push_back(const std::vector<_Tp>& elem);
void Mat::push_back(const Mat& m);
```

- elem : 행렬의 맨 마지막 행에 추가할 원소 데이터
- m : 행렬의 맨 마지막 행에 추가할 행렬. *this와 타입과 열 개수가 같아야 함

push_back 과는 반대로 행렬에서 맨 아래에 있는 행을 제거할 때는 Mat::pop_back()을 사용

```cpp
void Mat::pop_back(size_t nelems=1);
```

- nelems : 제거할 행 개수. *this 행렬의 행 개수보다 크면 안됨

예제 코드

```cpp
void MatOp7()
{
	Mat img1 = imread("lenna.bmp", IMREAD_GRAYSCALE);

	Mat img1f;
	img1.convertTo(img1f, CV_32FC1);

	uchar data1[] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 };
	Mat mat1(3, 4, CV_8UC1, data1);
	Mat mat2 = mat1.reshape(0, 1);

	cout << "mat1:\n" << mat1 << endl;
	cout << "mat2:\n" << mat2 << endl;

	Mat mat3 = Mat::ones(1, 4, CV_8UC1) * 255;
	mat1.push_back(mat3);
	cout << "mat1:\n" << mat1 << endl;

	mat1.resize(6, 100);
	cout << "mat1:\n" << mat1 << endl;
}
```

<img src="../../assets/images/110309.png" width="500px" height="500px" title="결과" alt="OP code"><img><br/>

## Vec 클래스

하나의 행으로만 이루어진 행렬은 행 벡터, 하나의 열로만 구성된 행렬은 열 벡터라고 부름

행 벡터와 열 벡터를 합쳐서 벡터 또는 벡터 행렬이라고 부름, 즉 벡터는 같은 자료형을 가진 원소 몇 개로 구성된 데이터 형식임

벡터 예제 코드

```cpp
Vec3b p1, p2(0, 0, 255);
```

p1는 (0, 0, 0), p2는 (0, 0, 255)로 생성됨

p1.val[0] = 100; 을 하게 되면 p1는 (100, 0, 0)을 가짐

p1[1] = 200; 으로도 바꿀 수 있음

```cpp
void VecOp()
{
	Vec3b p1, p2(0, 0, 255);
	p1.val[0] = 100;
	p1[1] = 200;

	cout << "p1: " << p1 << endl;
	cout << "p2: " << p2 << endl;

}
```

<img src="../../assets/images/110310.png" width="300px" height="300px" title="결과" alt="OP code"><img><br/>


## Scalar 클래스

Mat 클래스 다음으로 자주 사용되는 클래스임

4채널 이하의 영상에서 픽셀 값을 표현하는 용도로 자주 쓰임

```cpp
void ScalarOp() {
	Scalar gray = 128;
	cout << "gray: " << gray << endl;
	
	Scalar yellow(0, 255, 255);
	cout << "yellow: " << yellow << endl;

	Mat img1(256, 256, CV_8UC3, yellow);

	for (int i = 0; i < 4; i++)
		cout << yellow[i] << endl;
}
```

<img src="../../assets/images/110311.png" width="300px" height="300px" title="결과" alt="OP code"><img><br/>

## InputArray 클래스

InputArray 타입의 인자를 사용하는 함수를 자주 볼 수 있음

_InputArray 클래스는 다양한 타입으로부터 생성될 수 있는 인터페이스 클래스임

_InputArray::getMat() 함수를 사용하여 Mat 객체 타입 형태로 변환해서 사용해야 함

```cpp
Mat _InputArray::getMat(int idx=-1) const;
```

- idx : 참조할 행 번호. idx < 0 이면 행렬 전체를 참조함
- 반환값 : Mat 행렬 객체

예제 코드

```cpp
void printMat(InputArray _mat) {
	Mat mat = _mat.getMat();
	cout << mat << endl;
}

void InputArrayOp() {
	uchar data1[] = { 1, 2, 3, 4, 5, 6 };
	Mat mat1(2, 3, CV_8U, data1);
	printMat(mat1);

	vector<float> vec1 = { 1.2f, 3.4f, -2.1f };
	printMat(vec1);
}
```

<img src="../../assets/images/110312.png" width="300px" height="300px" title="결과" alt="OP code"><img><br/>

## OutputArray 클래스

많은 opencv 함수는 영상을 입력으로 받아 영상 처리를 수행하고, 그 결과를 다시 영상으로 생성하여 반환함

이 때 출력 영상을 함수의 return 구문으로 반환하는 것이 아니라 보통 OutputArray 클래스의 참조를 함수 인자로 사용하여 결과 영상을 전달함