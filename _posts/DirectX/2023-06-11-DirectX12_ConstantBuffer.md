---
title: "[DirectX12] DirectX12 공부노트 - ConstantBuffer를 사용하는 방법"
excerpt: "DirectX, Graphics"

categories: [DirectX, Graphics]
tags: [DirectX, Graphics]

toc: true
toc_sticky : true

date: 2023.06.11
last_modified_at : 2023.06.11
---
## **Directx 12 -3. ConstantBuffer**
---

* DirectX 12를 차근차근 공부해보자 시리즈의 대망의 세번째 포스팅

<br>

## **ConstantBuffer를 파이프라인에 탑재해보자**
---
![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/d910ad19-3677-456a-9598-9d4e51d28a0e)

<br>

* 프레임에 따라 사각형 정점의 위치를 offset만큼 이동시켜보자.

<br>

## **Header 파일에 추가할 내용과 생성자 부분의 수정**
---

* ConstantBuffer를 사용하기 위해 Struct를 선언하고 이를 변수로 선언 및 생성자에서 처리하도록 해보자.
* 참고로 앞에 "m_" 접두사가 붙은 변수들은 모두 클래스의 멤버변수이다.

<br>

```
 // D3D12Visualizer.h

    // 클래스 내에 struct를 새로 정의해주자.
    struct SceneConstantBuffer
    {
        XMFLOAT4 offset;
        float padding[60]; // Padding so the constant buffer is 256-byte aligned;
    };

    // 생성한 struct를 변수로 선언하여 클래스의 멤버 변수로 사용하도록 하자.
     SceneConstantBuffer m_constantBufferData;


// D3D12Visualizer.cpp

     D3D12Visualizer::D3D12Visualizer(UINT width, UINT height, std::wstring name) :
    DXSample(width, height, name),
    m_frameIndex(0),
    m_viewport(0.0f, 0.0f, static_cast<float>(width), static_cast<float>(height)),
    m_scissorRect(0, 0, static_cast<LONG>(width), static_cast<LONG>(height)),
    m_rtvDescriptorSize(0),
    m_constantBufferData{}
{

   m_vertexBufferCPU = nullptr;
   m_indexBufferCPU = nullptr;

    m_vertexBufferGPU = nullptr;
    m_indexBufferGPU = nullptr;
    m_vertexBufferUploader = nullptr;
    m_indexBufferUploader = nullptr;    
    
}
```

<br>

* LoadPipeline 함수 부분에 ConstantBuffer를 사용하기 위해 CBV(Constant Buffer View)와 Frame Resource를 위한 CPU Descriptor Handle을 만들어주자.
* 여기서 View란, GPU가 Resource를 받았을 때 어떤 방식으로 사용해야하는 지 설명해주는 작은 구조체를 의미한다.

<br>

```

void LoadPipeline()
{
    .
    .

        //----------------Constant Buffer-----------------//
                // Describe and create a constant buffer view(CBV) descriptor heap.
                // Flags indicate that this descriptor heap can be bound to pipeline
                // and that descriptors contained in it can eb referenced by ad root table
                D3D12_DESCRIPTOR_HEAP_DESC cbvHeapDesc = {};
                cbvHeapDesc.NumDescriptors = 1;
                cbvHeapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_SHADER_VISIBLE;
                cbvHeapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV;
                ThrowIfFailed(m_device->CreateDescriptorHeap(&cbvHeapDesc, IID_PPV_ARGS(&m_cbvHeap)));
            }
        //----------------Constant Buffer-----------------//

        //----------------Create frame resources -----------//
            {
                CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_rtvHeap->GetCPUDescriptorHandleForHeapStart());

                //Create a RTV for each frame.
                for(UINT n=0; n< FrameCount; n++)
                {
                    ThrowIfFailed(m_swapChain->GetBuffer(n, IID_PPV_ARGS(&m_renderTargets[n])));
                    m_device->CreateRenderTargetView(m_renderTargets[n].Get(), nullptr, rtvHandle);
                    rtvHandle.Offset(1, m_rtvDescriptorSize);
                }
        }
        
        //----------------Create frame resources -----------//
    .
    .

}

```

<br>

* ConstantBuffer를 직접 생성하고 여기에 데이터를 채워보자

```
// Load the sample assets.
void D3D12Visualizer::LoadAssets()
{
    // Create an empty root signature.
    {
        //---------------Constant Buffer--------------------//
                D3D12_FEATURE_DATA_ROOT_SIGNATURE featureData = {};

                //This ti the highest version the sample support. If CheckFeatureSupport succeeds,
                // the Highest version returned will not be greater than this.
                featureData.HighestVersion = D3D_ROOT_SIGNATURE_VERSION_1_1;
                if(FAILED(m_device->CheckFeatureSupport(D3D12_FEATURE_ROOT_SIGNATURE, &featureData, sizeof(featureData))))
                {
                    featureData.HighestVersion = D3D_ROOT_SIGNATURE_VERSION_1_0;
                }

                CD3DX12_DESCRIPTOR_RANGE1 ranges[1];
                CD3DX12_ROOT_PARAMETER1 rootParameters[1];

                ranges[0].Init(D3D12_DESCRIPTOR_RANGE_TYPE_CBV, 1, 0, 0, D3D12_DESCRIPTOR_RANGE_FLAG_DATA_STATIC);
                rootParameters[0].InitAsDescriptorTable(1, &ranges[0], D3D12_SHADER_VISIBILITY_VERTEX);

                // Allow input layout and deny unnecessary access to certain pipeline stages.
                D3D12_ROOT_SIGNATURE_FLAGS rootSignatureFlags =
                    D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT |
                    D3D12_ROOT_SIGNATURE_FLAG_DENY_HULL_SHADER_ROOT_ACCESS |
                    D3D12_ROOT_SIGNATURE_FLAG_DENY_DOMAIN_SHADER_ROOT_ACCESS |
                    D3D12_ROOT_SIGNATURE_FLAG_DENY_GEOMETRY_SHADER_ROOT_ACCESS |
                    D3D12_ROOT_SIGNATURE_FLAG_DENY_PIXEL_SHADER_ROOT_ACCESS;




                CD3DX12_VERSIONED_ROOT_SIGNATURE_DESC rootSignatureDesc;
            // rootSignatureDesc.Init(0, nullptr, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);
                rootSignatureDesc.Init_1_1(_countof(rootParameters), rootParameters, 0, nullptr, rootSignatureFlags);

                ComPtr<ID3DBlob> signature;
                ComPtr<ID3DBlob> error;
                ThrowIfFailed(D3DX12SerializeVersionedRootSignature(&rootSignatureDesc, featureData.HighestVersion, &signature, &error));
                ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_rootSignature)));
            }
        //---------------Constant Buffer--------------------//

        .
        .
        //----------------Constant Buffer------------------------------//
            
            const UINT constantBufferSize = sizeof(SceneConstantBuffer);

            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(constantBufferSize),
                D3D12_RESOURCE_STATE_GENERIC_READ,
                nullptr,
                IID_PPV_ARGS(&m_constantBufferGPU)));

            // Describe and create a constand buffer view.
            D3D12_CONSTANT_BUFFER_VIEW_DESC cbvDesc = {};
            cbvDesc.BufferLocation = m_constantBufferGPU->GetGPUVirtualAddress();
            cbvDesc.SizeInBytes = constantBufferSize;
            m_device->CreateConstantBufferView(&cbvDesc, m_cbvHeap->GetCPUDescriptorHandleForHeapStart());

            //// Map and initialize the constant buffer.
            //// We don't unmap this until the app closes.
            //// Keeping things mapped for the lifetime of the resource is okay
            CD3DX12_RANGE readRange(0, 0); // We don't intend to read from this resource on the cpu
            ThrowIfFailed(m_constantBufferGPU->Map(0, &readRange, reinterpret_cast<void**>(&m_pCbvDataBegin)));
            memcpy(m_pCbvDataBegin, &m_constantBufferData, sizeof(m_constantBufferData));

        //----------------Constant Buffer------------------------------//

```

<br>

* 아래는 실제로 매 틱당 constant buffer에 복사되는 값들을 나타내는 함수이다.
* m_constnatBufferData는 클래스의 멤버변수로, Update 함수가 call 될때마다 값이 변경되고 이를 GPU로 업로드 한다. 


<br>

```
void D3D12Visualizer::OnUpdate()
{
    const float translationSpeed = 0.005f;
    const float offsetBounds = 1.25f;
    m_constantBufferData.offset.x += translationSpeed;
    if(m_constantBufferData.offset.x > offsetBounds)
    {
        m_constantBufferData.offset.x =  -offsetBounds;
    }
    memcpy(m_pCbvDataBegin, &m_constantBufferData, sizeof(m_constantBufferData));
}
```

<br>

* 파이프라인에 ConstantBuffer를 묶는 방법이다.

<br>

```
void D3D12Visualizer::PopulateCommandList()
{
    .
    .

    //---------constant Buffer--------------------//

    ID3D12DescriptorHeap* ppHeaps[] = {m_cbvHeap.Get()};
    m_commandList->SetDescriptorHeaps(_countof(ppHeaps), ppHeaps);
    m_commandList->SetGraphicsRootDescriptorTable(0, m_cbvHeap->GetGPUDescriptorHandleForHeapStart());

    //---------constant Buffer--------------------//
    .
    .

```

* 지금까지 ConstantBuffer를 만들고 ConstantBuffer에 들어갈 데이터들을 모두 바인딩 해주었다.
* 이제는 Shader를 수정하여 ConstantBuffer를 사용하여 사각형을 이동할 수 있도록 HLSL 코드를 변경해보자.

<br>

```
// shader.hlsl

cbuffer SceneConstantBuffer : register(b0)
{
    float4 offset;
    float4 padding[15];
}


struct PSInput
{
    float4 position : SV_POSITION;
    float4 color : COLOR;
};

PSInput VSMain(float4 position : POSITION, float4 color : COLOR)
{
    PSInput result;

    result.position = position + offset;
    result.color = color;

    return result;
}

float4 PSMain(PSInput input) : SV_TARGET
{
    return input.color;
}


```




<br>

## **다음 포스팅에서는**
---

* 아래는 이후 포스팅에서 다룰 내용들에 대한 목록이다.

```
1) Texture 맵핑을 해보자.
2) FPS를 측정하고 이를 Window의 TitleBar에 띄워보자.
3) IMGUI를 통해 GUI를 붙여보자.
```

<br>

## **Sample Project** 
---
Github from : [DirectX 12 - Hello Trianglem Hello Constant Buffers](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld)

<br>


 ## ***Reference***
 ---
 * [Microsoft document](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-graphics)