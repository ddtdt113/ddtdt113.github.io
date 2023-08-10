---
title: "[DirectX12] DirectX12 공부노트 - FrameWork를 이해해보자"
excerpt: "DirectX, Graphics"

categories: [DirectX, Graphics]
tags: [DirectX, Graphics]

toc: true
toc_sticky : true

date: 2023.08.11
last_modified_at : 2023.08.11
---
## **Directx 12 -5. FrameWork를 이해해보자**
---

* DirectX 12를 차근차근 공부해보자 시리즈의 대망의 다섯번째 포스팅

<br>

# **FrameWork를 이해해보자**

## **왜 갑자기 FrameWork로 돌아왔나요?** 

![gif](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/e414235b-a522-4b68-973d-bf2d52ab08f6)
<br>

 ***[DirectX12 너무 어렵잖엉..ㅠㅠ]***

 <br>


 * 기존에는 마이크로 소프트에서 제공하는 샘플 코드를 통해 공부를 했습니다. 이를 통해 공부를 하기에는 막히는 부분이 굉장히 많았습니다.
 제가 생각하는 막히는 부분들은 다음과 같았습니다. 
 <br>
 
 ```
 1) 이론적인 진입장벽이 큽니다.
 2) DirectX 함수들에 대한 설명이 부족하다고 느꼈습니다.
 3) Sample Code를 바로 활용하기에는 제 능력이 부족합니다.
```

 위의 이유들로 인해 다른 Sample Code를 찾았습니다. 그러다가 많은 분들께 사랑받는 "Introduction to 3d programming with directX 12"책의 Sample Code를 공부해보기로 마음을 먹었습니다.

 * 소스코드 : https://github.com/yottaawesome/intro-to-dx12

<br>

# **FrameWork를 분석해보자**

## **기본적인 구조**
* 공부하려는 소스코드의 클래스 구조는 다음과 같습니다.


![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/b07e07c1-d698-44f0-ad87-7dc2fdba3dfc)

<br>

* d3dApp
    * d3dApp은 창(window)을 생성하기 위한 low level 기능들을 감싸고 있는 클래스입니다. 즉 DXGI과 관련된 내용과 DirectX12의 Command Allocator, Command Queue등을 이 곳에서 초기화하고 관리하게 됩니다. 또한 SwapChain, BackBuffer용 RenderTarget, DepthBuffer등을 모두 관리합니다. GPU로 command queue를 넘겨주는 FlushCommandQueue도 d3dApp에서 담당하네요. 추가적으로 GameTimer와 관련된 부분도 이 곳에서 담당합니다.

<br>

* Visualizer
    * Visualizer는 실제 샘플코드에는 없는 제가 만든 클래스입니다. 해당 클래스는 InitDirect3DApp를 복사해서 제가 이름만 바꾼 클래스입니다. 실제 샘플 클래스는 건들지 않고 공부를 하고 싶어서 이렇게 했네요. Visualizer 클래스는 d3dApp을 상속받은 클래스입니다. 즉 d3dApp위에서 작동하는 클래스이죠. 해당 클래스에서는 C++ Window 프로젝트인 Entry Point인 WinMain을 들고 있습니다. 이 프레임워크를 만든 분께서 창 생성 부분과 DirectX12의 초기화 부분들을 감추어 예제별 구체적인 세부사항에 집중할 수 있도록 구성하였다고 합니다. 이런 부분은 나중에 제가 프레임워크를 짤 때도 벤치마킹하고 싶은 부분입니다.

<br>

# **d3dApp은 어떻게 작동할까?** #
* d3dApp의 구조를 간단한 플로우 차트로 그려보았다.

![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/5b53080a-2970-44a7-95f6-f007aef3619a)

<br>

* 구조는 굉장히 심플합니다.. Game Loop안에서 Window message를 받으면 TranslateMessage를 통해 받은 msg 변수를 통해 미리 정의해둔 MsgProc 함수가 실행됩니다. MsgProc 함수는 GameTimer의 처리와 Mouse input이 들어왔을 때의 이벤트 처리를 담당하죠. 아래는 MsgProc함수의 소스코드이니 한번 읽어보시는 걸 추천드립니다.
<br>
```
LRESULT D3DApp::MsgProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam)
{
	switch( msg )
	{
	// WM_ACTIVATE is sent when the window is activated or deactivated.  
	// We pause the game when the window is deactivated and unpause it 
	// when it becomes active.  
	case WM_ACTIVATE:
		if( LOWORD(wParam) == WA_INACTIVE )
		{
			mAppPaused = true;
			mTimer.Stop();
		}
		else
		{
			mAppPaused = false;
			mTimer.Start();
		}
		return 0;

	// WM_SIZE is sent when the user resizes the window.  
	case WM_SIZE:
		// Save the new client area dimensions.
		mClientWidth  = LOWORD(lParam);
		mClientHeight = HIWORD(lParam);
		if( md3dDevice )
		{
			if( wParam == SIZE_MINIMIZED )
			{
				mAppPaused = true;
				mMinimized = true;
				mMaximized = false;
			}
			else if( wParam == SIZE_MAXIMIZED )
			{
				mAppPaused = false;
				mMinimized = false;
				mMaximized = true;
				OnResize();
			}
			else if( wParam == SIZE_RESTORED )
			{
				
				// Restoring from minimized state?
				if( mMinimized )
				{
					mAppPaused = false;
					mMinimized = false;
					OnResize();
				}

				// Restoring from maximized state?
				else if( mMaximized )
				{
					mAppPaused = false;
					mMaximized = false;
					OnResize();
				}
				else if( mResizing )
				{
					// If user is dragging the resize bars, we do not resize 
					// the buffers here because as the user continuously 
					// drags the resize bars, a stream of WM_SIZE messages are
					// sent to the window, and it would be pointless (and slow)
					// to resize for each WM_SIZE message received from dragging
					// the resize bars.  So instead, we reset after the user is 
					// done resizing the window and releases the resize bars, which 
					// sends a WM_EXITSIZEMOVE message.
				}
				else // API call such as SetWindowPos or mSwapChain->SetFullscreenState.
				{
					OnResize();
				}
			}
		}
		return 0;

	// WM_EXITSIZEMOVE is sent when the user grabs the resize bars.
	case WM_ENTERSIZEMOVE:
		mAppPaused = true;
		mResizing  = true;
		mTimer.Stop();
		return 0;

	// WM_EXITSIZEMOVE is sent when the user releases the resize bars.
	// Here we reset everything based on the new window dimensions.
	case WM_EXITSIZEMOVE:
		mAppPaused = false;
		mResizing  = false;
		mTimer.Start();
		OnResize();
		return 0;
 
	// WM_DESTROY is sent when the window is being destroyed.
	case WM_DESTROY:
		PostQuitMessage(0);
		return 0;

	// The WM_MENUCHAR message is sent when a menu is active and the user presses 
	// a key that does not correspond to any mnemonic or accelerator key. 
	case WM_MENUCHAR:
        // Don't beep when we alt-enter.
        return MAKELRESULT(0, MNC_CLOSE);

	// Catch this message so to prevent the window from becoming too small.
	case WM_GETMINMAXINFO:
		((MINMAXINFO*)lParam)->ptMinTrackSize.x = 200;
		((MINMAXINFO*)lParam)->ptMinTrackSize.y = 200; 
		return 0;

	case WM_LBUTTONDOWN:
	case WM_MBUTTONDOWN:
	case WM_RBUTTONDOWN:
		OnMouseDown(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
	case WM_LBUTTONUP:
	case WM_MBUTTONUP:
	case WM_RBUTTONUP:
		OnMouseUp(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
	case WM_MOUSEMOVE:
		OnMouseMove(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
    case WM_KEYUP:
        if(wParam == VK_ESCAPE)
        {
            PostQuitMessage(0);
        }
        else if((int)wParam == VK_F2)
            Set4xMsaaState(!m4xMsaaState);

        return 0;
	}

	return DefWindowProc(hwnd, msg, wParam, lParam);
}
```
<br>

* 자 이제 message를 받지 못했을 경우에는 3가지 함수가 실행되죠. Timer.Tick은 게임 타이머의 Tick(deltaTime)을 업데이트 해줍니다. 그리고 Update Constants 함수를 통해 앞으로 사용할 셰이더의 변수를 업데이트 해 주고, Draw 함수를 통해서 CommandList에 Draw하라고 명령을 던집니다. 여기서 Update함수와 Draw함수는 모두 가상함수로 되어있으며 해당 부분의 정의는 모두 상속받은 클래스인 Visualizer에 정의되어 있습니다. 


<br>

# 이번 포스팅에서 배운 점 #
* Sample Code의 FrameWork를 간단하게 분석해보았습니다. 이전에 공부한 마이크로 소프트의 Sample Code를 읽어본 경험 때문인지 다행히도 쉽게 분석이 되어서 기분이 좋습니다. 다음 포스팅에서는 3차원 Cube를 만들어보고 이를 회전시키는 코드를 작성해봅니다.

<br>


<br>


 ## ***Reference***
 ---
* Github from : https://github.com/yottaawesome/intro-to-dx12