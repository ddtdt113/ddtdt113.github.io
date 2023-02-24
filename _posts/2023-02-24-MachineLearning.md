---
title: "[MachineLearning] Machine Learning 공부노트 - Convoultion Neural Network의 개념"
excerpt: "ML"

categories:
    - MachineLearning
tags:
    - [MachineLearning, AI]


toc: true
toc_sticky : true

date: 2023.02.24
last_modified_at : 2023.02.24
---
## **정리에 앞서**
---
<br>
이번에 항상 공부하고 싶었던 Machine Learning을 공부할 기회가 생겨 공부한 내용들을 최대한 간략하게 적어보려한다.<br>

시리즈물로 작성해야지.
<br>

<br>

## **Convolution Neural Network이란**
---
Convolution Neural Network은 "이미지 처리"에 특화된 Neural Network이다.
주로 3가지 단계로 구성된다.
<br>

    1) Convolutional layer
    2) Activation function
    3) Pooling layer
<br>

Activation Function은 이후에 따로 다룰 예정이기에 이번에는 Convolutional Layer와 Pooling Layer에 대해 정리해보자.

![convnet](https://user-images.githubusercontent.com/41114834/221079662-e6980398-34a9-4dcc-b799-833339a22702.jpg)
    <center><span style="color:gray">[그림] Convolution Neural Network</span></center><br>

<br>

## **Convolutional layer(합성곱 층)이란**
---
Convolutional Layer의 역할은 한 줄로 정리하면 다음과 같다.
<br>

    이미지에서 특징(feature)을 추출하자!

<br>
특징점을 추출하는 것은 다음과 같은 조건들 때문에 진행한다.
<br>

        1) 이미지는 3d tensor이지만 일반적인 인공신경망의 입력값으로 집어 넣을 경우 1d로 변환된다.
        2) 1d로 변환되는 과정에서 이미지의 픽셀간의 공간적인 구조 정보가 유실된다. 
        3) 또한 같은 정보를 가진 이미지의 해상도가 커지면 커질수록 1d tensor로 변환 시 데이터의 변화가 커 인공신경망 학습과 예측 정확도를 떨어뜨린다.



![feature](https://user-images.githubusercontent.com/103714911/221081532-5d103311-62d2-41f7-be47-8cc47af74c23.png)
    <center><span style="color:gray">[그림] 특징점 추출</span></center><br>

특징점을 추출하는 과정은 필터(Filter) 혹은 커널(Kernel)의 개념이 등장한다. 필터나 커널은 다음과 같이 사용한다. 여기서는 커널이라는 표현을 사용하겠다. 커널은 한줄로 요약하면 다음과 같다. <br>

    이미지에서 특징을 뽑아내는 도구

<br>

커널은 ComputerVision에서 특정 이미지에 Edge detection을 하거나 Gaussian Blur를 적용시킬 때 사용한다.
대표적인 커널(혹은 Filter)은 Sobel Filter가 있다. 합성곱 신경망에서의 커널이 ComputerVision의 필터와 다른 점은 다음과 같다.<br>

    Computer Vision의 필터는 이미 값이 "채워져"있다.
    Conv의 커널은 "학습" 을 통해 "채워야"한다.

<br>

![1_xSzncJLaEjhoKC7LfM-cGA](https://user-images.githubusercontent.com/103714911/221086059-18f3c127-3e5e-412d-9563-adab0c38247c.png)
    <center><span style="color:gray">[그림] SobelFilter 예시</span></center><br>


Conv의 커널은 인공 신경망의 weight와 bias 값에 해당하는 부분이다. 즉, 학습을 통해 채워야 한다.
아래의 그림에서 볼 때 hidden layer 부분이 kernel이 된다.
<br>

![conv](https://user-images.githubusercontent.com/41114834/221084391-7ba486e5-f3cd-4652-bc23-273a0e6fba6d.png)
    <center><span style="color:gray">[그림] Conv2D</span></center><br>


합성곱층은 다음과 같이 사용한다.<br>
    
    1) Input 이미지에 지정된 사이즈의 커널을 곱한다.
    2) 해당 값을 Output Buffer쪽에 저장한다. 
위와 같은 연산 과정으로 인해 Input과 Output의 Tensor Dimension에 차이가 생긴다.
필요에 따라 패딩을 적용할 수 있으며 Stride 값에 따라 Kernel을 임의의 interval로 이동시켜 특징을 추출할 수 있다. 이에 대한 자세한 내용은 이곳을 참고하자.

<br>

[합성곱연산](https://wikidocs.net/64066)


![conv4](https://user-images.githubusercontent.com/41114834/221082755-9d89c60e-5bab-4d49-a6c0-891f6087d084.png)
    <center><span style="color:gray">[그림] 커널(Kernel)의 사용 방식</span></center><br>


## **Pooling layer(풀링층)이란**
---
풀링층의 역할을 조사해보니 다음과 같은 4가지 이유가 있다고 한다. [풀링층의 역할](https://supermemi.tistory.com/16)<br>

    1) Input Size를 줄여 Tensor Size를 줄인다.
    2) Overfitting을 조절하여 과적합(Overfitting)을 방지한다.
    3) Feature extraction이 향상된다.
    4) 일반화 성능을 향상시켜준다.

<br>

Pooling Layer(풀링층)은 주로 2가지 종류를 사용한다.<br>

    1) Average Pooling Layer
    2) Max Pooling Layer

<br>
Pooling Layer는 n X n 커널을 stride 단위로 이미지에 적용시킨 뒤 커널 안에 포함된 픽셀들 중 Max 혹은 Average 값을 추출해 Output값으로 갖는다.


![img1 daumcdn](https://user-images.githubusercontent.com/41114834/221087002-b0989e46-d8dd-4daa-b76c-a87bd96f4006.png)
    <center><span style="color:gray">[그림] Max or Average Pooling Layer</span></center><br>




### *Reference List*
---
* [Convolution Neural Network](https://wikidocs.net/64066)
* [인공신경망과 이미지](https://it-utopia.tistory.com/entry/%EB%B9%85%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B6%84%EC%84%9D%EA%B8%B0%EC%82%AC-%ED%95%A9%EC%84%B1%EA%B3%B1%EC%8B%A0%EA%B2%BD%EB%A7%9DConvolutional-Neural-Network)