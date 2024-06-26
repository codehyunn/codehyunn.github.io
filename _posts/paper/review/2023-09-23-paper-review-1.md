---
title: "[논문 리뷰] AlexNet : ImageNet Classification with Deep Convolutional Neural Networks"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Paper-Review
tags:
  - Paper
  - CNN
permalink: /paper-review/01/
last_modified_at: 2023-09-23
---
AlexNet에 대한 논문이다. 논문 링크는 [여기](https://proceedings.neurips.cc/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf) 👀🪄<br>
유명한 논문이지만, 제대로 읽어본 기억이 없어서 이번에 읽고 정리해봤다.

## | Introduction

물체 인식에서 이전까지의 모델들은 적은 데이터 셋을 활용한 간단한 테스크에 대해 잘 작동하는 모습을 보였다. 하지만 현실의 문제는 이보다 더 복잡하기 때문에 더 큰 데이터 셋과 더 좋은 성능을 가진 모델이 필요했다. 

모델은 더 큰 학습 능력과 사전 지식을 가지고 있어야 하는데, 이 조건을 만족하는 모델이 바로 `Convolutional Neural Networks (CNNs)`이다. CNN은 레이어의 깊이나 너비에 따라서 학습 능력을 조절할 수 있으며, 기존 Feed-forward Neural Networks에 비해 더 적은 connection과 파라미터만으로 학습이 가능하다는 장점이 있다. 하지만 CNN에 고해상도 이미지를 적용하는 것은 비용 문제가 발생했는데, 본 논문에서는 GPU 병렬 학습으로 해당 문제를 해결했다.

## | Dataset

ImageNet의 Subset으로 구성된 ILSVRC-2010 데이터 셋을 주로 사용했다.

입력 차원을 일정하게 맞추기 위해 이미지 해상도를 256x256로 통일하였고, 전체 픽셀에 대해 평균 값을 뺌으로써 centered 된 픽셀 값으로 학습을 진행했다.

## | Architecture

### Overall Architecture

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/7dd979c5-2b8c-49c0-bfe9-239716fa1d77){: .align-center}
    
총 8개의 layer(5개의 convolution layer와 3개의 fully connected layer)로 구성되어 있으며 본 논문에서 설명한 레이어 별 특징은 다음과 같다.

| 레이어 | 특징 |
| ----------- | --------------- |
| Layer 1       | input : 224 x 224 x 3 <br> kernel : 11 x 11 x 3 (96개, stride : 4) <br> Activation fn : ReLU <br> Response Normalization, Overlapping pooling |
| Layer 2       | kernel : 5 x 5 x 48 (256개) <br> Activation fn : ReLU <br> Response Normalization, Overlapping pooling |
| Layer 3       | kernel : 3 x 3 x 256 (384개) <br> Activation fn : ReLU |
| Layer 4       | kernel : 3 x 3 x 192 (384개) <br> Activation fn : ReLU |
| Layer 5       | kernel : 3 x 3 x 192 (256개) <br> Activation fn : ReLU <br> Overlapping pooling |
| Layer 6 (FC layer) | neuron : 4096개 <br> Activation fn : ReLU |
| Layer 7 (FC layer) | neuron : 4096개 <br> Activation fn : ReLU |
| Layer 8 (FC layer) | Activation fn : softmax <br> output : 1000 | <br>

### ReLU Nonlinearity

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/e150e747-85b9-4318-9e2a-1dfd131edb5b){: .align-center}


Non-saturating nonlinearity 함수(ReLU)가 Saturating nonlinearity 함수(sigmoid, tanh 등)에 비해 학습 속도가 빠르다. 그래서 본 논문에서는 거대한 모델과 데이터 셋에 대해서도 빠른 학습이 가능한 ReLU를 사용했다.

> **Saturating?** <br>
Saturating은 함수가 양/음의 무한대로 갈수록 0으로 수렴하는 것을 의미한다.
> <br>

### Training on Multiple GPUs
    
본 논문에서는 두 GPU를 병렬화하여 학습했다. 각 GPU에는 커널(혹은 뉴런)의 절반이 입력으로 주어진다. 그래서 두 GPU는 특정 레이어에서만 communicate 할 수 있다는 특징을 가진다.
<br>

### Local Response Normalization
    
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/4f5b65af-a0f5-4f47-a086-3d5b064eb1f2){:.align-center}

본 논문에서는 일반화(generatlization)을 돕는 Local normalization을 사용한다. Local normalization 은 일부 layer에 대해 ReLU 이후 적용하며, local contrast normalization과 달리 mean subtract 과정이 있다는 특징을 지닌다.

> **Input normalization이 필요하지 않은 이유?** <br>
Input normalizaiton은 보통 수렴을 방지하기 위해 사용하는 방법이지만 ReLU는 non-saturating nonlinearity 함수이기 때문에 필요하지 않다.
> <br>

### Overlapping Pooling
    
CNN의 Pooling layer는 같은 커널맵 안, 인접한 뉴런들의 출력을 압축한다. 기존에는 pooling unit이 겹치지 않도록 했지만, 본 논문에서는 pooling  unit을 겹치도록 설정했다. 이를 통해 과적합을 방지하는 효과를 낼 수 있었다.
<br>

## | Reducing Overfitting

상당한 과적합이 발생하여 해결 방안을 모색했다.

### Data Augmentation

데이터 수를 늘려 과적합을 방지하기 위해 연산량이 많지 않으면서 별도로 데이터를 저장하지 않아도 되는 방법을 적용했다. 

1. 이미지 이동(Image translation) 및 좌우 반전(Horizontal reflection)
2. 주성분 분석(PCA)을 이용한 RGB 값 변경
<br>

### Dropout
    
여러 모델의 예측 결과를 결합하면 성능이 올라가지만 보통 연산량도 증가한다는 단점이 있다. 이런 경우에 적용하기 좋은 방법이 바로 Dropout이다.

Dropout은 0.5의 확률로 은닉층 뉴런의 출력을 0으로 설정하는 것을 말한다. 0으로 설정된 뉴런은 forward와 back propagation 과정에 참여하지 않는다. 그 결과 뉴럴 네트워크는 매번 다른 구조를 가지겠지만, 서로 가중치는 공유한다. 이 방법은 Robust한 학습이 가능하다는 장점을 가진다.

본 논문에 따르면, Test 시에는 Dropout 없이 전체 뉴런을 사용했으며 결과에 1/2을 곱해주었다고 한다.


>**왜 가중치가 공유되는가** <br>
( *" ~ the neural network samples a different architecture, but **all these architecturses share weights**"* ) <br>
학습을 진행하면 매 iter마다 dropout하는 뉴런이 달라진다. 그런데 한 iter의 가중치는 다음 iter로 전달되어 업데이트 되니까 결국 모든 구조의 가중치들이 공유된다고 할 수 있다.
> <br>

## | Details of learning

최적화 알고리즘으로는 Stochastic Gradient Descent를 사용했으며, weight decay의 역할이 매우 중요하다는 점을 알아냈다. 
<br>

## | Results

- ILSRVC-2010, 2012에서 좋은 성능을 보였다.
- 각 GPU의 커널에서 다른 특징을 학습하는 양상을 확인 할 수 있다.    
- 이미지 속 물체의 위치에 상관없이 좋은 성능을 보인다.
<br>

## | Discussions

- 크고 깊은 CNN은 지도 학습(Supervised learning)에서 좋은 성능을 보인다.
- 하나의 레이어가 삭제되면 성능 저하가 나타나는 것으로 보아 깊이가 매우 중요한 요소임을 알 수 있다.
<br>

