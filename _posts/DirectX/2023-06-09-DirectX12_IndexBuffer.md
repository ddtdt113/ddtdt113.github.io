---
title: "[DirectX12] DirectX12 공부노트 - IndexBuffer를 사용하는 방법"
excerpt: "DirectX, Graphics"

categories: [DirectX, Graphics]
tags: [DirectX, Graphics]


toc: true
toc_sticky : true

date: 2023.06.09
last_modified_at : 2023.06.09
---
## **Directx 12 -2. IndexBuffer**
---

* DirectX 12를 차근차근 공부해보자 시리즈의 대망의 두번째 포스팅

<br>

## **IndexBuffer를 파이프라인에 탑재해보자**
---

![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/23398699-65ad-4f25-8c51-fadc0ddcb001)


* Sample Project의 Hello Triangle에 IndexBuffer를 붙여보자.
* IndexBuffer를 사용하여 사각형을 그려보자.

<br>

## **IndexBuffer과 VertexBuffer의 자원**
----

* IndexBuffer의 자원으로 다음과 같은 vertex와 index array을 만들자.

<br>

![image](https://github.com/ddtdt113/ddtdt113.github.io/assets/41114834/0d751573-177a-427c-8378-77302a955836)

<br>

```
     Vertex triangleVertices[] =
        {
            // x , y , z / r, g, ,b, a
            {{-1.0f,  1.0f, 0.0f},{1.0f, 1.0f, 1.0f, 1.0f}},
            {{ 1.0f,  1.0f, 0.0f},{1.0f, 0.0f, 1.0f, 1.0f}},
            {{ 1.0f, -1.0f, 0.0f},{1.0f, 1.0f, 1.0f, 1.0f}},
            {{-1.0f, -1.0f, 0.0f},{1.0f, 0.0f, 1.0f, 1.0f}},

        };

        //Create IndexBuffer
        unsigned int  triangleIndices[] =
        {          
            0,1,2,
            0,2,3
        };

```
<br>

* DirectX의 front face는 clock wise 방향이기 때문에 0-1-2, 0-2-3 순서로 IndexArray를 만들어주자.

<br>

## **IndexBuffer과 VertexBuffer의 업로드**
---

* VertexBuffer와 IndexBuffer를 업로드 하는 방식은 다음과 같다.

```
1) CPU에서 Vertex 혹은 Index 데이터를 들고 있을 Heap Buffer를 만든다. 
2) GPU가 CPU로 접근하여 Data를 읽도록 할 Heap Buffer(Uploader용 Buffer)를 만들어주자.
3) Subresource를 만든다.
4) 위 Heap Buffer과 SubResource를 commandList에 Job으로 던져 넣어주자. 

```
<br>

* 이때 Index, Vertex 데이터는 GPU에 올라갈 때까지 CPU에서 해제되면 안된다.

<br>

```
       const UINT vertexBufferSize = sizeof(triangleVertices);
        const UINT indexBufferSize = sizeof(triangleIndices);

        // VertexBuffer용 CPU용 heap 만들기
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
                D3D12_RESOURCE_STATE_COPY_DEST,
                nullptr,
                IID_PPV_ARGS(&m_vertexBufferGPU)));


        // VertexBuffer용 Uploader heap 만들기
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
                D3D12_RESOURCE_STATE_GENERIC_READ,
                nullptr,
                IID_PPV_ARGS(&m_vertexBufferUploader)));

             NAME_D3D12_OBJECT(m_vertexBufferGPU);

             D3D12_SUBRESOURCE_DATA vertexData = {};
             vertexData.pData = triangleVertices;
             vertexData.RowPitch = vertexBufferSize;
             vertexData.SlicePitch = vertexData.RowPitch;

             UpdateSubresources<1>(m_commandList.Get(), m_vertexBufferGPU.Get(), 
                                   m_vertexBufferUploader.Get(), 0,0,1, &vertexData );
             m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_vertexBufferGPU.Get(), 
             D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_VERTEX_AND_CONSTANT_BUFFER));

        // VertexBufferView 세팅
             m_vertexBufferView.BufferLocation = m_vertexBufferGPU->GetGPUVirtualAddress();
             m_vertexBufferView.StrideInBytes = sizeof(Vertex);
             m_vertexBufferView.SizeInBytes = vertexBufferSize;
        

    // IndexBuffer용 CPU용 heap 만들기
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_DEFAULT),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(indexBufferSize),
                D3D12_RESOURCE_STATE_COPY_DEST,
                nullptr,
                IID_PPV_ARGS(&m_indexBufferGPU)));

    // IndexBuffer용 Uploader heap 만들기
            ThrowIfFailed(m_device->CreateCommittedResource(
                &CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
                D3D12_HEAP_FLAG_NONE,
                &CD3DX12_RESOURCE_DESC::Buffer(indexBufferSize),
                D3D12_RESOURCE_STATE_GENERIC_READ,
                nullptr,
                IID_PPV_ARGS(&m_indexBufferUploader)));

            NAME_D3D12_OBJECT(m_indexBufferGPU);

            D3D12_SUBRESOURCE_DATA indexData = {};
            indexData.pData = triangleIndices;
            indexData.RowPitch = indexBufferSize;
            indexData.SlicePitch = indexData.RowPitch;

    // IndexBufferView 세팅        
            m_indexBufferView.BufferLocation = m_indexBufferGPU->GetGPUVirtualAddress();
            m_indexBufferView.Format = DXGI_FORMAT_R32_UINT;
            m_indexBufferView.SizeInBytes = indexBufferSize;
        
    // Subresource 생성 및 commadList에 Job 던져주기
            UpdateSubresources<1>(m_commandList.Get(), m_indexBufferGPU.Get(), m_indexBufferUploader.Get(), 0, 0, 1, &indexData);
            m_commandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_indexBufferGPU.Get(), D3D12_RESOURCE_STATE_COPY_DEST, D3D12_RESOURCE_STATE_INDEX_BUFFER));


        // commandList를 닫아주고 실행시켜주자!
        ThrowIfFailed(m_commandList->Close());
        ID3D12CommandList* ppCommandLists[] = { m_commandList.Get() };
        m_commandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);
```

## **PopulateCommandList의 수정**
---
* 위 같이 세팅을 해도 IndexBuffer가 적용되지 않는다면 지금 이 부분을 빼먹은 것이다.
* DrawInstanced로 되어있는 부분을 DrawIndexedInstanced로 변경해줘야 최종적으로 IndexBuffer를 DirectX12 파이프라인에 적용시킬 수 있다. 이 부분이 왜 그런지 모른다면 반드시 DirectX11의 파이프라인을 확인해보는 것을 권장한다. DirectX12보다 더 직관적이다.

<br>

```
// VertexBufferView를 통해 Pipeline에 VertexBuffer 던져주기
    m_commandList->IASetVertexBuffers(0, 1, &m_vertexBufferView);

// VertexBufferView를 통해 Pipeline에 IndexBuffer 던져주기
    m_commandList->IASetIndexBuffer(&m_indexBufferView);

// 삼각형을 그릴 방식 세팅
    m_commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

// 6은 IndexBuffer의 element 수 이다.
    m_commandList->DrawIndexedInstanced(6, 1, 0, 0, 0);

```

<br>

## **다음 포스팅에서는**
---

* 아래는 이후 포스팅에서 다룰 내용들에 대한 목록이다.

```
1) Texture 맵핑을 해보자.
2) Update함수를 통해 셰이더를 제어해보자. 이는 ConstantBuffer등을 통해 제어할 예정이다.
3) FPS를 측정하고 이를 Window의 TitleBar에 띄워보자.
4) IMGUI를 통해 GUI를 붙여보자.
```


## **Sample Project** 
---
Github from : [DirectX 12 - Hello Triangle](https://github.com/microsoft/DirectX-Graphics-Samples/tree/master/Samples/Desktop/D3D12HelloWorld)

<br>



 ## ***Reference***
 ---
 * [Microsoft document](https://learn.microsoft.com/en-us/windows/win32/direct3d12/direct3d-12-graphics)