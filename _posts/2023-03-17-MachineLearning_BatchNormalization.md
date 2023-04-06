---
title: "[MachineLearning] Machine Learning 공부노트 - BatchNormalization의 이해"
excerpt: "ML"

categories:
    - MachineLearning
tags:
    - [MachineLearning, AI]


toc: true
toc_sticky : true

date: 2023.04.06
last_modified_at : 2023.04.06
---
## **BatchNormalization의 이해**
---

* BatchNormalization의 사용 목적과 연산 방식에 대해 알아보자. 
<br>

## **Index**
---
* Batch Normalization이란 무엇인가?
* 수식으로 이해하는 Batch Normalization
* Batch Normalization의 Gamma, Beta의 의미
* Test 단계에서의 Batch Normalization 계산

<br>

## **BatchNormalization이란 무엇인가?**
---
* Layer로 들어가는 입력값이 너무 퍼지거나 너무 좁아지지 않게 해주는 인공 신경망 기법
<br>

![image](https://user-images.githubusercontent.com/41114834/230259587-a3abec88-7882-47e4-807d-6464d44b0f3c.png)
<br>

* Batch Normalization 다음 시점에서 주로 적용된다.
    * mini-batch가 layer를 통과하여 획득한 weighted sum값이 activativation function에 들어가기 전 시점

![image](https://user-images.githubusercontent.com/41114834/230259769-1b1487bf-373f-4a8b-9c1b-205babdc2c86.png)
<br>

<br>

## **수식으로 이해하는 Batch Normalization**
---
* Batch hNormalization Equation
![image](https://user-images.githubusercontent.com/41114834/230259938-384abf6e-1055-440c-af71-2604ea12c4fb.png)
<br>

* 수식이 가지는 의미는 다음과 같다.
    - 입력값(x)는 **평균이 0에 분산이 1인** 값으로 정규화(Normalization)된다.

<br>

### **수식을 적용해보자**
* 다음과 같이 5000개의 Data Set을 50개의 mini batch로 분리해서 어떠한 Neural Network Layer를 통과하고 BatchNormalization Layer로 들어간다고 해보자.
![image](https://user-images.githubusercontent.com/41114834/230260154-c566b97c-397e-4629-a5c4-5440726ee2bb.png)
<br>

* 그러면 mini-batch가 다음과 같이 Batch Normalization Layer로 들어가게 될 것이다.
![image](https://user-images.githubusercontent.com/41114834/230260258-7f44fbf8-4d03-40eb-b9fa-2e10da7f25b9.png)
<br>

* Batch Normalization 내부에서는 위에서 보았던 수식을 통해 입력값들을 정규화(Normalization)한다.
이때 Gamma, Beta는 딥러닝 네트워크를 트레이닝하는 과정에서 채워진다. 즉, Convolution layer에서의 weight, bias와 같은 역할을 하게 된다.
![image](https://user-images.githubusercontent.com/41114834/230260870-0843c4a4-b83c-4fb6-8595-f236eb1768c8.png)
<br>

* 트레이닝이 끝나면 고정된 Gamma, Beta값을 획득하게 된다.
 ![image](https://user-images.githubusercontent.com/41114834/230261379-c4a00fa2-1be9-49aa-94f0-59559efa6a7f.png)
<br>

<br>

## **Batch Normalization의 Gamma, Beta의 의미**
* Batch Normalization에서의 Gamma와 Beta는 non-linearity를 유지하고 vanishing gradient를 해결하기 위해 존재한다.
![image](https://user-images.githubusercontent.com/41114834/230261682-771d16b7-5738-4482-88e3-b8eb9f6a4a04.png)
<br>

![image](https://user-images.githubusercontent.com/41114834/230261720-457dc44f-6d62-4fb1-ac7a-aa066dd2fabe.png)
![image](https://user-images.githubusercontent.com/41114834/230261746-109e964e-3024-4cd1-bc97-fa7bc749d7fe.png)
![image](https://user-images.githubusercontent.com/41114834/230261768-9e3277b3-a710-44c5-b857-edc6a3da67a2.png)
<br>

<br>

## **Test 단계에서의 Batch Normalization 계산**
---
* Training 단계에서는 mini-batch별 평균값과 분산값을 구할 수 있었다. 하지만 Test단계에서는 단일 입력값이 들어올 경우도 있을 텐데 이때 Gamma와 Beta값은 어떻게 구할까?
![image](https://user-images.githubusercontent.com/41114834/230262171-c5b16e27-9ca3-44b0-b18b-87f8a4fdc998.png)
<br>

* 이는 간단하게 구할 수 있다. 바로 Training 단계에서 구한 mini-batch별 평균값들의 평균, 분산값들의 분산을 가져다가 쓰면 된다.
![image](https://user-images.githubusercontent.com/41114834/230262391-ea61eeb3-f904-4cbd-95df-f2b6138cdf84.png)

* 위 내용은 2D Tensor같이 단일 채널인 경우에만 가능한 연산인데, 이를 3D Tensor로 확장하면 어떻게 될까?
이는 간단하게 확장된다. Gamma와 Beta, 평균, 분산값을 채널별로 들고 있으면 된다.
![image](https://user-images.githubusercontent.com/41114834/230262504-fddcbc79-bbaa-47e9-aa19-57011ffce3e6.png)


### *Reference List*
---
* [혁펜하임 - 가장 깔끔한 배치 정규화 설명](https://www.youtube.com/watch?v=m61OSJfxL0U&t=911s)
* [Pytorch - BatchNorm2d](https://pytorch.org/docs/stable/generated/torch.nn.BatchNorm2d.html)