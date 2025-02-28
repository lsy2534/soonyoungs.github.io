---
layout: post
title: Deep Learning CNN
date: 2025-02-26 00:00 +0800
last_modified_at: 2025-02-26 00:08:25 +0800
tags: [AI, Deep Learnig, CNN ]
categoie: [AI]
topic:  true

---
**Deep learning CNN**!
{: .message }

## 개요
기존의 인공 신경망(ANN)이 이미지 및 공간적 데이터 처리에 비효율적이었기 때문에 이에 합성곱 신경망이 탄생하게 되었다.

이유 
1. 고정된 입력 크기와 연산량 문제
 - 기존 인공 신경망(ANN)은 완전 연결 구조를 사용하기 때문에, 입력 이미지의 크기가 조금만 커져도 매우 많은 가중치와 연산량이 필요하게 된다.
 - 이런 구조는 고해상도 이미지에 적용하기 어렵고, 학습 과정에서 과적합(overfitting) 문제도 심각해집니다.

2. 공간적 구조(Spatial Structure) 보존의 필요성
이미지 데이터는 *픽셀간의 공간적 관계(경계 패턴, 윤곽선), 인공신경만은 모든 픽셀을 독립적인 입력으로 처리하기 때문에 공간적 패턴을 인식하는데 비효율적이었다.

3. CNN의 핵심 아이디어는 생물학적 신경망, 특히 인간의 시각 피질에서 영감을 받아 만들었다.
* 1960년대 허블 & 비셀(Hubel & Wiesel) 연구에 따르면, 인간의 시각 피질에는 특정 패턴(예: 선, 가장자리, 질감 등)을 감지하는 뉴런들이 존재합니다.
* CNN은 이를 모방하여 계층적으로 특징을 학습하는 구조(Convolution → Pooling → Fully Connected Layers)를 설계했습니다.

합성곱 층은 단순히 필터의 모음이다. 필터에 저장된 값은 훈련하는 동안 신경망이 학습하는 가중치 이다.

합성곱 층(Conv Layer) 여러 개의 필터가 동시에 작동하는 측이며, 각각의 필터가 서로 다른 특징을 감지한다. 필터가 많을수록 더 다양한 특징을 학습할 수 있으며 결국 합성곱 층은 여러 개의 필터를 한한꺼번에 적용하는 곳이기 때문에 필터의 모음이라고 부르는것이다. CNN(합성곱)에서 필터의 가중치는 처음에는 랜덤한 값으로 설정 된다. 처음에는 필터가 아무런 의미 있는 패털을 감지하지 못하고 단순히 랜험한 값이 들어 있기 때문에, 필터를 적용해도 이미지에서 중요한 특징을 추출하는 역을 못 하다가 학습이 진행 될 수록 필터가 점점 의미 있는 역할을 하게 된다.

비유를 하자면 초반에는 아무것도 모르던 아이가 점점 사물을 구별하는 것과 비슷하다 


CNN에서 합성곱(Convolution) 연산을 할 때, 필터(커널)가 이미지 위를 이동하며 특징을 추출한다.
이때 **"얼마나 이동할 것인지"**와 **"이미지를 어떻게 다룰 것인지"**를 결정하는 두 가지 요소가 **스트라이드(Stride)와 패딩(Padding)**

```
💡 직관적인 비유
✅ Stride = 사람이 계단을 오를 때 몇 칸씩 오를지 정하는 것

한 칸씩 가면 천천히 이동 (Stride = 1)
두 칸씩 가면 빠르게 이동 (Stride = 2)
✅ Padding = 종이에 그림을 그릴 때 여백을 추가하는 것

여백 없이 그리면 그림이 종이 끝에서 잘릴 수도 있음 (패딩 없음)
여백을 추가하면 끝까지 다 그릴 수 있음 (패딩 있음)

특징 맵(Feature Map)
컨볼루션 연산을 수행한 결과
필터가 입력 데이터에서 어떤 특징을 감지했는지를 나타내는 출력 값

폴링(Pooling)
풀링은 특징 맵의 크기를 중이는 과정
불필요한 정보는 버리고, 중요한 정보만 유지하면서 계산량을 줄여주는 역할

📌 대표적인 풀링 기법

맥스 풀링(Max Pooling) → 해당 영역에서 가장 큰 값을 선택
평균 풀링(Average Pooling) → 해당 영역의 평균 값을 선택
```

CNN을 이용한 이미지 분류(Image Classification) 과정

1. 입력 데이터 (이미지) 준비
2. 합성곱(Convolution) & ReLU 적용
 - **"필터"**를 이용해 이미지의 특징을 추출
 - 필터를 이동하면서 특징 맵 생성
 - ReLu 활성화 함수 적용
   -  음수를 0으로 바꿔서 비선형성을 추가
3. 폴링(Pooling)으로 특징 맵 크기 축소
4. 완전 연결 신경망(FC Layer) 적용
 - Flatten(평탄화) : 2D 특징 맵을 1D 벡터로 변환
 - 완전 연결층(FC Layer): 신경망을 통해 분류 작업 진행

 5. 소프트맥스(Softmax)로 클래스 활률 계산
 6. 출력

 파이썬 + TensorFlow/Keras를 활용해서 간단한 이미지 분류 만들기

 MNIST(손글씨 숫자 데이터)를 활용하여 데이터셋 만들기

 ```
import tensorflow as tf
from tensorflow import keras
import matplotlib.pyplot as plt
import numpy as np

# MNIST 데이터셋 불러오기와 튜플로 변환환
mnist = keras.datasets.mnist
(x_train, y_train), (x_test, y_test) = mnist.load_data()

# 데이터 정규화 (0~255 범위를 0~1로 변환)
x_train, x_test = x_train / 255.0, x_test / 255.0

# 입력 데이터를 CNN에 맞게 차원 추가 (채널 추가)
x_train = x_train.reshape(-1, 28, 28, 1)
x_test = x_test.reshape(-1, 28, 28, 1)


# reshape(-1, 28, 28, 1)
# CNN 모델은 4D 텐서(배치, 높이, 너비, 채널)를 입력으로 받음.
# 흑백 이미지니까 채널 1개(Grayscale) 추가함.
```

# CNN 모델 만들기

합성곱 신경망(CNN) 모델을 만들기
```
model = keras.Sequential([
    #첫 번째 합성곱 층 (Conv2D) + 활성화 함수 (ReLU)
    keras.layers.Conv2D(32, (3,3), activation='relu', input_shape=(28, 28, 1)),
    keras.layers.MaxPooling2D((2,2)),  # 맥스 풀링(Max Pooling)

    # 두 번째 합성곱 층 (Conv2D) + 활성화 함수 (ReLU)
    keras.layers.Conv2D(64, (3,3), activation='relu'),
    keras.layers.MaxPooling2D((2,2)),  # 다시 맥스 풀링

    keras.layers.Flatten(),  # 2D 데이터를 1D로 변환
    keras.layers.Dense(128, activation='relu'),  # 완전 연결층(Dense Layer)
    keras.layers.Dense(10, activation='softmax')  #  출력층 (10개의 클래스)
])

# 모델 컴파일
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

# 모델 구조 확인
model.summary()

```

