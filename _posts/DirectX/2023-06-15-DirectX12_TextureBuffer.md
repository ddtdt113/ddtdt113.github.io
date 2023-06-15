---
title: "[DirectX12] DirectX12 공부노트 - TextureBuffer를 사용하는 방법"
excerpt: "DirectX, Graphics"

categories: [DirectX, Graphics]
tags: [DirectX, Graphics]

toc: true
toc_sticky : true

date: 2023.06.15
last_modified_at : 2023.06.15
---
## **Directx 12 -4. TextureBuffer**
---

* DirectX 12를 차근차근 공부해보자 시리즈의 대망의 네번째 포스팅

<br>

## **TextureBuffer를 파이프라인에 탑재해보자**
---
![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/2c50ff46-1467-4a23-9e3d-4a50e9eba2a9)

<br>

## **TextureBuffer의 탑재 순서** 
---
* LoadPipeLine()
    * Texture용 SRV(Shader Resource View) Descriptor 선언
    * Texture용 Sampler Descriptor 선언
* LoadAsset()
    * TextureBuffer를 Cpu -> GPU로 바인딩 시킬 Buffer 생성 및 CommandList에 업로드
* PopulateCommandList()
    * Texture Buffer Heap 업로드
    * Texture Buffer용 Root DescriptorTable 생성
* GenerateTextureData()
    * Texture Data 생성


<br>



## **코드를 통해 알아보자**
---

* Texture용 SRV(Shader Resource View) Descriptor 선언
* Texture용 Sampler Descriptor 선언

<br>

```
D3D12_FEATURE_DATA_ROOT_SIGNATURE featureData = {};

        //This is the highest version the sample support. If CheckFeatureSupport succeeds,
        // the Highest version returned will not be greater than this.
        featureData.HighestVersion = D3D_ROOT_SIGNATURE_VERSION_1_1;
        if(FAILED(m_device->CheckFeatureSupport(D3D12_FEATURE_ROOT_SIGNATURE, &featureData, sizeof(featureData))))
        {
            featureData.HighestVersion = D3D_ROOT_SIGNATURE_VERSION_1_0;
        }

        CD3DX12_DESCRIPTOR_RANGE1 ranges[1];
        ranges[0].Init(D3D12_DESCRIPTOR_RANGE_TYPE_SRV, 1, 0, 0, D3D12_DESCRIPTOR_RANGE_FLAG_DATA_STATIC);

        CD3DX12_ROOT_PARAMETER1 rootParameters[1];
        rootParameters[0].InitAsDescriptorTable(1, &ranges[0], D3D12_SHADER_VISIBILITY_PIXEL);

        D3D12_STATIC_SAMPLER_DESC sampler = {};
        sampler.Filter = D3D12_FILTER_MIN_MAG_MIP_POINT;
        sampler.AddressU = D3D12_TEXTURE_ADDRESS_MODE_BORDER;
        sampler.AddressV = D3D12_TEXTURE_ADDRESS_MODE_BORDER;
        sampler.AddressW = D3D12_TEXTURE_ADDRESS_MODE_BORDER;
        sampler.MipLODBias = 0;
        sampler.MaxAnisotropy = 0;
        sampler.ComparisonFunc = D3D12_COMPARISON_FUNC_NEVER;
        sampler.BorderColor = D3D12_STATIC_BORDER_COLOR_TRANSPARENT_BLACK;
        sampler.MinLOD = 0.0f;
        sampler.MaxLOD = D3D12_FLOAT32_MAX;
        sampler.ShaderRegister = 0;
        sampler.RegisterSpace = 0;
        sampler.ShaderVisibility = D3D12_SHADER_VISIBILITY_PIXEL;


        CD3DX12_VERSIONED_ROOT_SIGNATURE_DESC rootSignatureDesc;
        rootSignatureDesc.Init_1_1(_countof(rootParameters), rootParameters, 1, &sampler, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

        ComPtr<ID3DBlob> signature;
        ComPtr<ID3DBlob> error;
        ThrowIfFailed(D3DX12SerializeVersionedRootSignature(&rootSignatureDesc, featureData.HighestVersion, &signature, &error));
        ThrowIfFailed(m_device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_rootSignature)));
```

<br>


* TextureBuffer를 Cpu -> GPU로 바인딩 시킬 Buffer 생성 및 CommandList에 업로드

<br>

```
  /// Describe and create a Texture2D
             D3D12_RESOURCE_DESC textureDesc = {};
             textureDesc.MipLevels = 1;
             textureDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
             textureDesc.Width = TextureWidth;
             textureDesc.Height = TextureHeight;
             textureDesc.Flags = D3D12_RESOURCE_FLAG_NONE;
             textureDesc.DepthOrArraySize = 1;
             textureDesc.SampleDesc.Count = 1;
             textureDesc.SampleDesc.Quality = 0;
             textureDesc.Dimension = D3D12_RESOURCE_DIMENSION_TEXTURE2D;

             ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
                D3D12_HEAP_FLAG_NONE,
                &textureDesc,
                D3D12_RESOURCE_STATE_COPY_DEST,
                nullptr,
                IID_PPV_ARGS(&m_texture)));

            const UINT64 uploadBufferSize = GetRequiredIntermediateSize(m_texture.Get(), 0 ,1);

            // Create the GPU upload Buffer
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(uploadBufferSize),
                D3D12_RESOURCE_STATE_GENERIC_READ,
                nullptr,
                IID_PPV_ARGS(&m_textureUploadHeap)));
            
            // Copy data to the intermediate upload heap and then schedule a copy
            // from the upload heap to the Texture2D.
            std::vector<UINT8> texture = GenerateTextureData();

            D3D12_SUBRESOURCE_DATA textureData = {};
            textureData.pData = &texture[0];
            textureData.RowPitch = TextureWidth * TexturePixelSize;
            textureData.SlicePitch = textureData.RowPitch * TextureHeight;

            UpdateSubresources(m_commandList.Get(), m_texture.Get(), m_textureUploadHeap.Get(), 0, 0, 1, &textureData);
            m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_texture.Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE));

            //Describe and create a SRV for the texture
            D3D12_SHADER_RESOURCE_VIEW_DESC srvDesc = {};
            srvDesc.Shader4ComponentMapping = D3D12_DEFAULT_SHADER_4_COMPONENT_MAPPING;
            srvDesc.Format = textureDesc.Format;
            srvDesc.ViewDimension = D3D12_SRV_DIMENSION_TEXTURE2D;
            srvDesc.Texture2D.MipLevels = 1;
            m_device->CreateShaderResourceView(m_texture.Get(), &srvDesc, m_srvHeap->GetCPUDescriptorHandleForHeapStart());
```

<br>

* Texture Buffer Heap 업로드
* Texture Buffer용 Root DescriptorTable 생성

<br>

```
    ID3D12DescriptorHeap* ppHeaps[] = {m_srvHeap.Get()};
    m_commandList->SetDescriptorHeaps(_countof(ppHeaps), ppHeaps);
    m_commandList->SetGraphicsRootDescriptorTable(0, m_srvHeap->GetGPUDescriptorHandleForHeapStart());

```

<br>

* Texture Data 생성

<br>

```
std::vector<UINT8> D3D12Visualizer::GenerateTextureData()
{
    const UINT rowPitch = TextureWidth * TexturePixelSize;
    const UINT cellPitch = rowPitch >> 3;               // the width of a cell in the checkboard texture.
    const UINT cellHeight = TextureWidth >> 3;          // the height of a cell in the checkboard texture.
    const UINT textureSize = rowPitch * TextureHeight;

    std::vector<UINT8> data(textureSize);
    UINT8* pData = &data[0];

    for(UINT n=0; n < textureSize; n += TexturePixelSize)
    {
        UINT x= n % rowPitch;
        UINT y = n & rowPitch;
        UINT i = x / cellPitch;
        UINT j = y / cellHeight;

        if(i % 2 == j % 2)
        {
            pData[n] = 0x00; // R
            pData[n + 1] = 0x00; // G
            pData[n + 2] = 0x00; // B
            pData[n + 3] = 0xff; // A
        }
        else
        {
            pData[n] = 0xff; // R
            pData[n + 1] = 0xff; // G
            pData[n + 2] = 0xff; // B
            pData[n + 3] = 0xff; // A

        }

    }

    return data;
}
```

<br>

* Shader 코드 수정

<br>

```
struct PSInput
{
    float4 position : SV_POSITION;
    float2 uv : TEXCOORD;
};

Texture2D g_texture : register(t0);
SamplerState g_sampler : register(s0);

PSInput VSMain(float4 position : POSITION, float2 uv : TEXCOORD)
{
    PSInput result;

    result.position = position;
    result.uv = uv;

    return result;
}

float4 PSMain(PSInput input) : SV_TARGET
{
    return g_texture.Sample(g_sampler, input.uv);

}
```

<br>


## **다음 포스팅에서는**
---

* 아래는 이후 포스팅에서 다룰 내용들에 대한 목록이다.

```
1) FPS를 측정하고 이를 Window의 TitleBar에 띄워보자.
2) IMGUI를 통해 GUI를 붙여보자.
```

<br>

## **Sample Project** 
---
Github from : [DirectX 12 - Hello Triangle, Hello Texture](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld)

<br>


 ## ***Reference***
 ---
 * [Microsoft document](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-graphics)