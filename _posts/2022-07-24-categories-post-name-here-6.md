---
title: "[👏] 아주대 학습 공동체 스터디 (opencv) 2"
excerpt: "기본 자료형 클래스"

categories:
  - 학습공동체
tags:
  - [C++, opencv]

permalink: /categories6/post-name-here-6/

toc: true
toc_sticky: true

date: 2022-10-31
last_modified_at: 2022-10-31
---

## 기본 자료형 클래스

### Point_클래스

- 2차원 평면 위에 있는 점의 좌표를 표현하는 템플릿 클래스

어떤 자료형으로 좌표를 표현할 것인지를 명시해야 함. ex) Point_<int>

opencv에서는 자주 사용하는 자료형에 대하여 Point_ 클래스 이름을 재정의해서 제공함. ex) Point2i

```cpp
Point pt1; // pt1 = (0,0)
pt1.x = 5; pt1.y = 10; // pt1 = (5,10)
Point pt2(10,30); // pt2 = (10,30)
```
연산 수행도 가능

```cpp
Point pt3 = pt1 + pt2; // pt3 = [15, 40]
Point pt4 = pt1 * 2; // pt4 = [10, 20]
int d1 = pt1.dot(pt2); // d1 = 350
bool b1 = (pt1 == pt2); // b1 = false
```

다음은 pt1과 pt2변수에 저장된 좌표를 화면에 출력하는 코드 예제

```cpp
cout << "pt1: " << pt1 << endl;
cout << "pt2: " << pt2 << endl;
```

실제로 이 코드를 실행하면 콘솔 창에 다음과 같이 점의 좌표가 출력됨

pt1: [5, 10]

pt2: [10, 30]

### Size_ 클래스

- 영상 또는 사각형 영역의 크기를 표현할 때에 사용
- 사각형 영역의 가로와 세로 크기를 나타내는 width와 height 멤버 변수를 가짐

Point_ 클래스와 마찬가지로 재정의 된 함수 이름이 있음

```cpp
Size sz1, sz2(10,20); // sz1 = [0 x 0], sz2 = [10 x 20]
sz1.width = 5; sz1.height = 10; // sz1 = [5 x 10]
```

사칙 연산 가능

```cpp
// sz1 = [5 x 10], sz2 = [10 x 20]
Size sz3 = sz1 + sz2; // sz3 = [15 x 30]
Size sz4 = sz1 * 2; // sz4 = [10 x 20]
int area1 = sz4.area(); // area1 = 200
```

정보 출력

```cpp
cout << "sz3: " << sz3 << endl
cout << "sz4: " << sz4 << endl
```

출력 결과

sz3: [15 x 30]

sz4: [10 x 20]

### Rect_ 클래스

- OpenCV에서 사각형의 위치가 크기 정보를 표현할 대에는 Rect_ 클래스를 사용

```cpp
Rect rc1; // rc1 = [0 x 0 from (0, 0)]
Rect rc2(10, 10, 60, 40); // rc2 = [60 x 40 from (10, 10)]
```

사칙 연산 가능

```cpp
// rc1 = [0 x 0 from (0, 0)], rc2 = [60 x 40 from (10, 10)]
Rect rc3 = rc1 + Size(50, 40); // rc3 = [50 x 40 from (0, 0)]
Rect rc4 = rc2 + Point(10, 10); // rc4 = [60 x 40 from (20, 20)]
```

- Rect_ 객체끼리 서로 논리 연산 수행 가능
- 두 개의 사각형 객체 끼리 & 연산을 수행하면 두 사각형이 교차하는 사각형 영역을 반환
- 두 사각형 객체 끼리 | 연산을 수행하면 두 사각형을 모두 포함하는 최소 크기의 사각형을 반환함

```cpp
// rc3 = [50 x 40 from (0, 0)], rc4 = [60 x 40 from (20, 20)]
Rect rc5 = rc3 & rc4; // rc5 = [30 x 20 from (20, 20)]
Rect rc6 = rc3 | rc4; // rc6 = [80 x 60 from (0, 0)]
```

같은 내용은 생략하겠음

### RotatedRect 클래스

- 회전된 사각형을 표현하는 클래스
- 좌표 중심 좌표를 나타내는 center
- 가로 및 세로 크기를 나타내는 size
- 회전 각도 정보를 나타내는 angle 

중심 좌표가 (40, 30), 크기는 40 x 20, 시계 방향으로 30만큼 회전된 사각형 객체

```cpp
RotatedRect rr1(Point2f(40, 30), Size2f(40, 20), 30.f);
```

만약 회전된 사각형 객체의 네 꼭지점 좌표를 알고 싶다면

```cpp
Point2 pts[4];
rr1.points(pts);
```

꼭지점 좌표가 pts 배열에 저장됨

경우에 따라서 회전된 사각형을 감싸는 최소 크기의 사각형 정보가 필요 -> 바운딩 박스

```cpp
Rect br = rr1.boundingRect();
```

### Range 클래스

- 범위 또는 구간을 표현하는 클래스
- 범위의 시작과 끝을 나타내는 start와 end 멤버 변수를 가지고 있음

```cpp
Range r1(0, 10); // 0부터 9까지의 범위를 표현하고 10은 포함하지 않음
```

### String 클래스

- opencv에서 문자열을 다룰 때, 문자열을 지정하여 구분하고 출력하는 기능 제공하는 함수

```cpp
typedef std::string String;
```

예시

```cpp
String str1 = "Hello";
String str2 = "world";
String str3 = str1 + " " + str2; // str3 = "Hello world"
```

내용이 같은지 비교하는 예제

```cpp
bool ret = (str2 == "WORLD");
```

== 연산자는 대소문자를 구분함

특정한 형식의 문자열을 만들고 싶다면 format()함수 사용
```cpp
String format(const char* fmt, ...);
```
- fmt : 형식 문자열
- ... : 가변 인자
- 반환값 : 지정한 형식으로 생성된 문자열

만일 test01.bmp, test02.bmp, test03.bmp 세 개의 테스트 파일을 불러오고 싶을 때

```cpp
Mat imgs[3];
for (int i = 0; i < 3; i++){
  String filename = format("test%02d.bmp", i + 1);
  imgs[i] = imread(filename);
}
```
