---
title: "[MachineLearning] LibTorch - 기본 예제"
excerpt: "MachineLearning, LibTorch"

categories:
    - MachineLearning, LibTorch
tags:
    - [MachineLearning, LibTorch]


toc: true
toc_sticky : true

date: 2023.05.04
last_modified_at : 2023.05.04
---
## **LibTorch - 기본 예제**
---

* LibTorch를 사용해보며 배운 내용들을 적어보자.
* 최대한 간결한 코드로 Sample Code를 작성하자.

<br>



## **Tensor의 선언** 
---
* 가장 간단하게 Tensor를 선언해보자
```
int main()
{
	cout<< "Hello world!" << endl;
	
// 1) ----- Tensor 초기화
	torch::Tensor a = torch::zeros(5).cuda();
	cout << "a : " << a << endl;

// 1) ---- 선언한 Tensor 값 변경
	a[0] = 10.0;
	cout<< "a modified : " << a <<endl;


	return 0;
}

// output)
----------------
Hello world!
a :  0   0  0   0   0 [ CUDAFloatType{5} ]
a modified :  10  0  0  0  0 [ CUDAFloatType{5} ]
```


<br>

## ***Reference***
 ---
* [PyTorch](https://pytorch.org/cppdocs/notes/tensor_creation.html)