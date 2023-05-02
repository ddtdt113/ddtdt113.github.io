---
title: "[DirectX12] DirectX12 공부노트 - Hello Window"
excerpt: "DirectX, Graphics"

categories:
    - DirectX, Graphics
tags:
    - [DirectX, Graphics]


toc: true
toc_sticky : true

date: 2023.05.03
last_modified_at : 2023.05.03
---
## **Directx 12 -1.Hello Window**
---

* DirectX 12를 차근차근 공부해보자 시리즈의 대망의 첫번째 포스팅
<br>
<br>


## **Sample Project** 
---
Github from : [DirectX 12 - Hello Window](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld)

<br>

## **Hello Window!**
---
![image](https://user-images.githubusercontent.com/41114834/235722862-6f540b09-c777-4cc5-ab33-98a7242b87ed.png)
<br>

* DirectX12에서의 Hello World는 무엇일까? 그건 바로 Window를 띄우는 것이다! Hello Window!
* 어떻게 창이 띄워지는 지 간략하게 알아보자.

<br>

## **Hello Window 프로젝트 구조**
---
```
D3D12HelloWindow
    | ---- D3D12HellowWindow.h / cpp
    | ---- d3dx12.h
    | ---- DxSample.h /cpp
    | ---- DxSampleHelper.h
    | ---- Main.cpp
    | ---- stdafx.h / cpp
    | ---- Win32Application.h /cpp

```

## **Hello Window 함수 흐름**
---
### ***Entry Point***
```
    - Main.cpp - WinMain()
```
<br>

![image](https://user-images.githubusercontent.com/41114834/235724859-23e74743-b994-4326-b1a3-b449f964ba7c.png)

* WINAPI : 윈도우 용 콘솔 앱을 제작할 때 사용하는 Windows api
 * 콘솔앱이 실질적으로 시작되는 부분!
    * HINSTANCE : 프로그램의 자체의 주소!
    * [LPSTR](https://espada4897.wordpress.com/2014/12/23/lpstr%EA%B3%BC-lpctstr%EC%97%90-%EA%B4%80%ED%95%B4%EC%84%9C/) : *char
    * [nCmdShow](https://stackoverflow.com/questions/15240036/what-is-ncmdshow)
* 해당 부분의 역할
    * Window Size, 타이틀바에 들어갈 메세지를 설정한 D3D12HelloWorld 오브젝트를 만든다. (띄울 창(window)의 탄생!)

<br>

### ***Win32Application::Run(&sample, hInstance, nCmdShow)***
---
* 위 함수는 다음과 같은 역할을 한다.
    * 프로세스 초기화
    * 띄울 창(window) 관련 변수들 초기화 및 오브젝트 생성
    * Pipeline 초기화, Asset 초기화
    * MainLoop를 통해 매 틱당 연산을 진행

<br>

1) 프로세스 초기화
    ![image](https://user-images.githubusercontent.com/41114834/235729467-8fd79139-86e6-4b0c-9f31-f90d1202f76c.png)
<br>

2) 띄울 창(window) 관련 변수들 초기화 및 오브젝트 생성
    ![image](https://user-images.githubusercontent.com/41114834/235729673-87d25216-37ae-43cb-a3b6-1e934c953392.png)
<br>

    ![image](https://user-images.githubusercontent.com/41114834/235729872-652491fa-f228-4983-93c5-028d179a6645.png)
<br>

3) Pipeline 초기화, Asset 초기화
     ![image](https://user-images.githubusercontent.com/41114834/235730380-ad7e69de-d4eb-4d1e-b2f5-6444f65734c8.png)

<br>

4) MainLoop를 통해 매 틱당 연산을 진행
      
      ![image](https://user-images.githubusercontent.com/41114834/235729838-9aee95ca-abad-41de-bc78-6a1c0d868644.png)

      * 실질적으로 이 부분에서는 [WM_QUIT](https://learn.microsoft.com/ko-kr/windows/win32/winmsg/wm-quit) 메세지가 나타났는지 매 틱 확인하는 역할을 한다.
      * 이때 [Callback함수](https://namu.wiki/w/callback%20%ED%95%A8%EC%88%98)인  Win32Application::WindowProc가 DispatchMessage 이후 매 틱당 호출되게 된다. 자세한 내용은 Callback함수를 참고하자.

<br>


### ***Win32Application::WindowProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)***
---
* 위 함수에서는 다음과 같은 역할을 한다.
    * 입력값에 대한 이벤트 처리
        * WM_CREATE, WM_KEYDOWN, WM_KEYUP, WM_PAINT, WM_DESTROY
    * OnUpdate(), OnRender()를 통해 매 틱당 parameter 업데이트, 업데이트된 결과 Render를 수행한다.

    ![image](https://user-images.githubusercontent.com/41114834/235732247-77499df9-a79b-4bc5-aab6-ff5dcbb44b87.png)

<br>

## ***OnRender(), OnUpdate()에서는 구체적으로 어떤 역할을 할까?***
---
 * 이는 다음 포스팅에서 알아보자

 ## ***Reference***
 ---
 * [Microsoft document](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-graphics)