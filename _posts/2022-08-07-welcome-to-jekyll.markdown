---
title:  "2022-08-07"
date:   2022-08-07 15:04:23
categories: [자율주행자동차]
tags: [교내]
---

### 자율주행자동차 lane detection

동관 휴게실이 문을 닫아 8월 5일에 동관 104호?를 빌려 청소를 진행하였다.

창고로 쓰던 장소라 그런지 청소가 매우 힘들었다....


![image3.png](../../images/080706.jpg)


오늘 동관에서 lane detection을 sliding window로 해보려 했는데 동관이 공사중이라 원천관에서 진행하였다 ㅠ

일단 내가 lane detection을 맡아 관련하여 오늘 적용했던 것들을 정리하자면

`허프 변환과 sliding window가 있었다.`

1. 허프 변환
- 영상신호처리 과목에서 배웠으며 허프 변환으로 lane을 detection할 수 있었음.
- 좌표축을 다르게 하여 선을 점으로 표현하여 lane을 쉽게 추출할 수 있게 됨.

![image.png](../../images/080701.jpg)


2. sliding window 방법
- window를 sliding하여 물체가 있으면 그 부분을 표시함
- 사물 인식에 많이 쓰는 기법
- lane detection에서도 많이 쓰는 기법

<p align="center">
<img src="../../images/080703.gif">
</p>

### 허프 변환

영상신호처리 과목에서 배웠던 허프 변환을 통해 lane detection 해보았다. 

```python
import cv2
import numpy as np
import math

def line_detection():
    file = "./load.jpg"
    
    # load img
    src = cv2.imread(file,cv2.IMREAD_GRAYSCALE)
    
    if src is None:
        print("Error opening Image!")
        print('Usage: hough_lines.py [image_name -- default ' + file + '] \n')
        return -1
    
    print(src.shape)
    src2 = src[250:,250:]
    src3 = src[250:,250:]
    # Canny는 gradient를 계산함
    dst = cv2.Canny(src2, 50, 200, None, 3)
    
    # BGR로 display하기 위해 변환
    cdst = cv2.cvtColor(dst, cv2.COLOR_GRAY2BGR)
    cdstP = np.copy(cdst)
    
    lines = cv2.HoughLines(dst, 1, np.pi / 180, 150, None, 0, 0)
    
    if lines is not None:
        for i in range(0, len(lines)):
            rho = lines[i][0][0]
            theta = lines[i][0][1]
            a = math.cos(theta)
            b = math.sin(theta)
            x0 = a * rho
            y0 = b * rho
            pt1 = (int(x0 + 1000*(-b)), int(y0 + 1000*(a)))
            pt2 = (int(x0 - 1000*(-b)), int(y0 - 1000*(a)))
            
            cv2.line(cdst,pt1,pt2,(0,0,255),3,cv2.LINE_AA)
            
            
    linesP = cv2.HoughLinesP(dst, 1, np.pi / 180, 50, None, 50, 10)
    black_img = np.zeros((src3.shape[0],src3.shape[1]), dtype=np.uint8)
    black_img = cv2.cvtColor(black_img, cv2.COLOR_GRAY2BGR)
    if linesP is not None:
        for i in range(0, len(linesP)):
            l = linesP[i][0]
            cv2.line(cdstP, (l[0],l[1]), (l[2], l[3]), (0,0,255), 3, cv2.LINE_AA)
            cv2.line(black_img, (l[0],l[1]), (l[2], l[3]), (0,0,255), 3, cv2.LINE_AA)
            print(l[0],l[1])
    print(linesP)
    
    black_img = black_img[:,:,2]

    cv2.imshow('load',src)
    cv2.imshow("Dected Lines (in red) - Standard Houghh Line Transform", cdst)
    cv2.imshow("Dected Lines (in red) - Probabilistic Line Transform", cdstP)
    
    M = cv2.moments(black_img)
    if M['m00'] != 0:
        x = int(M['m10']/M['m00'])
        y = int(M['m01']/M['m00'])
    else:
        x = 0
        y = 0
    cv2.imshow('line',black_img)
    cv2.imwrite('line.jpg',black_img)
    black_img = cv2.cvtColor(black_img, cv2.COLOR_GRAY2BGR)
    cv2.circle(black_img, (x,y), 5, (0, 255, 0), 1)
    print(x, y)
    cv2.imshow('result',black_img)
    
    cv2.waitKey(0)

    cv2.destroyAllWindows() 
    return 0

line_detection()
```

결과는 다음과 같다.

다음과 같은 도로 사진이 있을 때 

![image2.png](../../images/080704.png)

빨간색 선으로 표시된 결과 화면을 보면 도로를 선으로 인식했다는 것을 볼 수 있다. 

허프 변환을 통해 도로 lane을 따올 수 있었다.

`이 방법이 좋은지는 구동해보면서 확인해봐야겠다.`

### Sliding Window

<iframe id="video" width="600" height="350" src="../../images/080702.mp4" frameborder="0">
</iframe>


sliding window 방식으로 차선이 있을 시 사각형으로 표현할 수 있도록 구현하였다.

(개인적으로 sliding window 구현하는게 매우 힘들었다.... pixel 위치랑 웬만한 cv2 함수는 거의 알아야 구현가능했었다.)


``` python
def pyramid(self, image, scale=1.5, minSize=(30,30)):
	yield image

        while True:
            w = int(image.shape[1] / scale)
            image = imutils.resize(image, width=w)

            if image.shape[0] < minSize[1] or image.shape[1] < minSize[0]:
                break

            yield image

def sliding_window(self, image, stepSizeX, stepSizeY, windowSize):
	for y in range(0, image.shape[0], stepSizeX):
            for x in range(0, image.shape[1], stepSizeY):
                yield (x,y,image[y:y+windowSize[1], x:x+windowSize[0]])

def sliding(self):
        ret, thresh_cv = cv2.threshold(self.crop_image, 127, 255, cv2.THRESH_BINARY)
        (winW, winH) = (30, 30)
        thresh_cv = cv2.cvtColor(thresh_cv,cv2.COLOR_GRAY2BGR)
        real_image = thresh_cv.copy()
        stepSizeX = 20
        stepSizeY = 20
        start_x = 0 
        start_y = 0
        for (x, y, window) in self.sliding_window(thresh_cv, stepSizeX=stepSizeX, stepSizeY=stepSizeY, windowSize=(winW,winH)):
            if window.shape[0] != winH or window.shape[1] != winW:
                continue

            if np.sum(thresh_cv[y:y+winH, x:x+winW,0])/255 > 500:
                if self.cnt == 0:
                    start_x = x
                    start_y = y
                    self.cnt = self.cnt + 1
                elif start_y == y :
                    self.cnt = self.cnt + 1
                else :
                    cv2.rectangle(real_image, (start_x, start_y), (start_x + stepSizeY * (self.cnt-1) + winW, start_y + winH), (0,255,0), 2)
    
                    cv2.circle(real_image, (int((2 * start_x + stepSizeY * (self.cnt-1) + winW)/2), int((2 * start_y + winH)/2)), 3, (0,0,255),1)
                    
                    self.cnt = 0

            else :
                if self.cnt != 0 :

                    cv2.rectangle(real_image, (start_x, start_y), (start_x + stepSizeY * (self.cnt-1) + winW, start_y + winH), (0,255,0), 2)
                    cv2.circle(real_image, (int((2 * start_x + stepSizeY * (self.cnt-1) + winW)/2), int((2 * start_y + winH)/2)), 3, (0,0,255),1)
                    self.cnt = 0
        cv2.imshow("window",real_image)
```


결과화면을 보면 다음과 같다.

차선을 sliding window로 표시하여 나타낼 수 있었다.

<iframe id="video" width="600" height="350" src="../../images/080705.mp4" frameborder="0">
</iframe>

### 결론
- 허프 변환으로 lane detection하는 것은 효과가 좋지 못했다.
	- 관련 자료는 내일 업로드할 예정
- sliding window로 lane detection하는 것이 더 좋아보였다. 
	- 무엇보다도 안정적으로 차선을 표시할 수 있었다.

**해야할 것**
- filtering을 통해 노이즈 제거가 가능한 지 구현해봐야겠다.
- lane detection을 통하여 차량 위치를 어떻게 표현할지 방법을 생각해봐야겠다.

