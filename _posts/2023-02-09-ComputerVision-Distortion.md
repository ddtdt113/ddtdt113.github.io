---
title: "[ComputerVision] Camera Calibration - Radial Distortion vs Tangential Distortion"
excerpt: "blank"

categories:
    - ComputeVision
tags:
    - [Camera, ComputerVision]


toc: true
toc_sticky : true

date: 2023.02.09
last_modified_at : 2023.02.09
---
## ***#*** ***Radial Distortion*** vs ***Tangential Distortion***
---
<span style="color:gray">***'Single Camera Calibration with OpenCV Library and C++'***</span><br>
OpenCV를 통해서 Camera Calibration을 공부하고 있다.
공부를 하면서 몰랐던 개념에 대해 간단하게 정리하고 싶어 블로그에 글을 남기게 되었다.
<br>

## ***Radial Distortion은 무엇인가?***
---

* ***Radial Distortion : 방사왜곡***
    * 볼록렌즈의 굴절률에 의해서 발생
    * 이미지의 왜곡 정도가 중심에서의 거리에 의해 결정되는 특징
    * 일반적인 경우에는 왜곡 계수(k1,k2,k3) 중 k1,k2만으로도 보정이 가능하지만 광각 렌즈같이 왜곡이 심한 렌즈의 경우에는 k3까지 사용
<br>


<br>
![radial](https://user-images.githubusercontent.com/41114834/217764277-44095c90-9c34-462e-8fdc-ee529a779b17.png)
<br>
    <center><span style="color:gray">[그림] Radial Distortion</span></center><br>

## ***Equation - Radial disotrtion***
-----
![image](https://user-images.githubusercontent.com/41114834/217772441-d629c031-8a96-4c1a-a750-ee9a50418f76.png)


---

<br>

## ***Tangential Distortion은 무엇인가?***
---
* ***Tangential Disortion : 접선왜곡***
    * 카메라 제조 과정에서 카메라 렌즈와 이미지센서(CMOS)의 수평이 발생하지 않거나 센서와 카메라 렌즈의 Center가 달라 발생됨
    * 타원형 형태로 왜곡 분포가 달라짐


<br>
![tangential](https://user-images.githubusercontent.com/41114834/217755726-ede4cedc-e17b-4bb7-a720-c358ac2ed59e.png)
<br>
    <center><span style="color:gray">[그림] Tangential Distortion</span></center>
  <br>

## ***Equation - Tangential distortion***
---
![image](https://user-images.githubusercontent.com/41114834/217772585-154688bc-1767-449c-baec-ce16adbb4bc9.png)


 

<br>
   
#### ***Reference***
---
* Radial, tangential distortionimage source : https://kr.mathworks.com/help/vision/ug/camera-calibration.html
* 카메라 왜곡보정 - 이론 및 실제 : https://darkpgmr.tistory.com/31
* [OpenCV] 카메라 보정(#3 렌즈왜곡, 방사왜곡, 접선왜곡) : http://carstart.tistory.com/181
* 어안 보정 기본 사항 : https://kr.mathworks.com/help/vision/ug/fisheye-calibration-basics.html
* 수식 작성법 : https://velog.io/@d2h10s/LaTex-Markdown-%EC%88%98%EC%8B%9D-%EC%9E%91%EC%84%B1%EB%B2%95