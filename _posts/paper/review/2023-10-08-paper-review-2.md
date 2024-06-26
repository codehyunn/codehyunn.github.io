---
title: "[논문 리뷰] VGGNet : Very Deep Convolutional Networks for Large-Scale Image Recognition"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Paper-Review
tags:
  - Paper
  - CNN
permalink: /paper-review/02/
last_modified_at: 2023-10-08
---

논문 링크는 [여기](https://browse.arxiv.org/pdf/1409.1556.pdf)👀🪄

**🧚🏻‍♀️짧게 정리해보자면 . . .**<br>
저자는 ConvNets의 성능을 높이기 위한 방법으로 **깊이 증가**를 선택했다. <br>이때 **3x3 Convolution layer**를 사용했는데, 같은 크기의 receptive field를 가지더라도 더 적은 수의 파라미터를 사용하는 효과를 얻을 수 있었다.
{:.notice--blog-theme}


## | Introduction

Convolutional Networks (ConvNets)은 고해상도 이미지 및 비디오 인식에서 좋은 성능을 보였고, 컴퓨터 비전 분야에서 중요한 위치를 갖게 되었다. 이에 AlexNet의 구조를 발전시켜 성능을 높이려는 시도가 있었다.

1. 첫 번째 convolution layer의 receptive window size와 stride 줄이기
2. 다양한 크기의 이미지에 대해서 학습하고 테스트하기

본 논문에서는 <mark style='background-color: #E9ECED'>깊이</mark>에 집중한다. 다른 파라미터를 고정한 채 <mark style='background-color: #E9ECED'>convolution layer (3x3)</mark>를 추가하는 방식으로 깊이를 증가시켰고, 이렇게 얻은 구조로 실험을 진행했다.

결과에 대해 짧게 언급하는데, 모든 부분(ILSVRC classification, localisation, 다른 데이터 셋에 적용) 에서 성능이 뛰어났다고 한다.
{:.notice} 

## | ConvNet Configurations
Network의 Architecture와 파라미터 설정 등에 대해 언급하는 부분이다.
{:.notice} 

### 01. Architecture
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/81725ef2-e548-4f1f-9bae-ef8de6fcbdb4){: .align-center}

#### Input

<mark style='background-color: #E9ECED'>24x224 RGB 이미지</mark>를 입력으로 한다. 전처리는 학습 데이터 셋에 대해서만 진행했는데, 전체 픽셀에 대해 RGB 별로 각 평균값을 빼는 <mark style='background-color: #E9ECED'>mean subtraction</mark>을 적용했다.

#### Convolution Layer

사용한 커널의 사이즈는 <mark style='background-color: #E9ECED'>3x3</mark>이다. stride는 1로 고정했고, 입력 해상도를 유지하기 위해 spatial padding을 적용했다. 추가적으로 한 구조에 대해서는 1x1도 사용했다고 한다. 

**Spatial Padding?**<br>
Convolution layer를 통과한 output feature map의 size가 줄어들지 않고 input feature map의 size와 똑같이 유지되도록 하는 것을 의미한다.
{:.notice}

#### Fully Connected Layer (FC Layer)

3개의 FC Layer와 Softmax Layer를 가진다. 앞의 두 레이어는 4096개의 채널을, 마지막 레이어는 클래스 개수에 해당하는 1000개의 채널을 가진다.

#### 추가적인 특징
활성화 함수로는 ReLU를 사용했다. AlexNet에서 등장했던 Local Response Normalization(LRN)은 많은 메모리나 시간을 필요로 하지만 막상 성능 향상은 잘 나타나지 않아 적용하지 않았다.

### 02. Configurations

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/ceb25d05-f1de-40ee-a3ec-f4ca5d8dab68){: .align-center}

A-E의 깊이는 서로 다르지만 모두 같은 형태를 가진다. 
아래의 표를 살펴보면, 전부 64의 convolution layer width로 시작하여 `'Feature map 2배 → max-pooling'` 과정을 반복하다가 512로 끝나는 것을 확인할 수 있다.

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/64e7febd-c5c4-464d-acb7-e1286a9dc6da){: .align-center}

다른 얕고 넓은 모델(144M)에 비해 파라미터 수가 적다. 

### 03. Discussion
#### 3x3 Convolution layer 사용

3x3 convolution layer 자체는 큰 receptive field를 갖지 않지만, 여러 개를 함께 사용함으로써 <mark style='background-color: #E9ECED'>큰 receptive field의 효과</mark>를 냈다. Convolution layer를 여러 개 사용함에 따라 Rectification layer (ReLU)도 여러 개 사용하였는데, 덕분에 예측의 정확도를 높였고 <mark style='background-color: #E9ECED'>파라미터 수를 줄였다.</mark>

#### 1x1 Convolution layer 사용

C에서 사용됐다. Receptive field에 영향을 미치진 않았지만 <mark style='background-color: #E9ECED'>비선형성을 증가</mark>시켰다.

**GoogleNet vs. VGGNet**<br>
GoogleNet과 VGGNet 모두 deep convolution network에 small convoultion filter를 적용한 구조이다.
저자는 VGGNet에 비해 GoogleNet의 구조가 더 복잡하며 첫 번째 layer에서 feature map의 해상도가 더 급격하게 줄어든다고 언급했다. 기분 탓일지 모르겠지만 왠지 VGGNet이 낫다고 하는 느낌이 들어서 재밌게 읽었다. 다음엔 GoogLeNet 논문 읽어보고 VGGNet과 비교해봐야겠다.꒰⍤꒱
{:.notice} 

## | Classification Framework
모델 학습과 평가에 대해 설명하는 부분이다.
{:.notice} 
### 01. Training

#### Settings
최적화 알고리즘으로는 모멘텀이 적용된 미니배치 경사하강법을 사용했으며, dropout과 weight decay으로 규제(regularization)를 적용했다. 파라미터는 아래의 표와 같이 설정했다.

| 파라미터 | 설정 |
| :------------------ | :----------------------------- | 
| Batch size | 256 |
| Momentum | 0.9 |
| Dropout<br>(dropout ratio) | 0.5 |
| Weight decay<br>(L2 penalty multiplier) | 5e-4 |
| Learning rate | 1e-2<br>** 정확도가 증가하지 않으면 1/10씩 감소시킴 (총 3번 감소) |
| Epochs | 74 | <br>

<br>
저자는 초기 가중치가 좋지 않다면 학습이 종료될 수 있기 때문에 <mark style='background-color: #E9ECED'>가중치 초기화(weight initialization)</mark>가 중요하다고 했다. 그래서 깊은 네트워크로 학습을 진행할 때에는 이미 학습된 가중치로 초기화하고 학습을 진행했다. 이때 사용된 가중치는 랜덤으로 초기화하여 학습해도 영향이 없는 얕은 구조(A)로 학습하여 얻었다.

랜덤으로 초기화된 가중치의 경우, normal distribution(mean 0, std 1e-2)을 따라 샘플링 되었다고 한다. 그리고 본 논문에서는 미리 학습한 가중치를 받아와서 사용하는 방식으로 실험을 진행했는데, 저자가 이런 pre-training 없이 가중치를 초기화할 수 있는 방법을 나중에 발견했다고 언급했다. 언급된 방법(논문)은 바로 Xavier initialization! 어떤 방식인지 궁금하다꒰⍤꒱
{:.notice}


마지막으로 데이터다. 입력이 224x224로 고정되어 있기 때문에 Rescaled image를 random crop 하여 224x224로 만들었고, random horizontal flip과 random RGB shift를 적용하여 데이터를 증강했다.

왜 매번 헷갈리는지 모르겠지만, horizontal flip = 좌우반전이다.
{:.notice} 

#### Training Image Size
Training image scale S에 대해선 두 가지 접근법을 사용했다.

| 접근법 | 설명 |
| :------ | :--- |
| Single-scale training | S를 256, 384로 고정하여 실험하며, S=256의 가중치로 초기화했다. |
| Multi-scale training | 각 이미지의 S를 [256, 512] 사이의 값으로 랜덤하게 설정한다.<br>Scale jittering을 통한 학습 데이터 셋 augmentation으로 볼 수 있으며, single-scale model(S=384)의 가중치로 초기화했다. |


### 02. Testing

Test scale Q는 S와 다를 수 있으며 본 논문에선 성능을 높이기 위해 S마다 <mark style='background-color: #E9ECED'>다양한 Q</mark>로 실험했다. 

평가할 때에는 FC layer를 convolution layer로 변경하여 <mark style='background-color: #E9ECED'>fully convolutional net</mark>을 만들었다. 이때 첫 번째 레이어는 7x7 convolution layer로, 나머지 두 레이어는 1x1 convolution layer로 바꿨다. Output으로는 `input의 해상도 * 클래스 수 (채널)`의 형태를 가진 class score map이 주어지며, class score map을 평균내어 최종 score를 구할 수 있다.

Fully convolution net은 전체 이미지에 대해 적용되기 때문에 multiple crops를 샘플링해서 사용하지 않고(=AlexNet), large set 하나를 사용했다. 또, Multi-crop evaluation과 dense evaluation은 ConvNet에 crop을 적용할때 padding을 채우는 방법이 다른데, 그렇기 때문에 서로 상호 보완적이다.

3개의 scale에 대해 50 crop씩, 총 150 crop으로 평가를 진행했다.

**Multi-crop evaluation vs. Dense evaluation**<br>
Multi-crop evaluation은 test image를 자르고, 모델을 이용하여 각각을 분류한 다음, 그 결과들을 추합해서 최종 결과를 내는 방법이다. 이때, 자른 부분에 대한 padding은 0으로 채운다.<br>~~Dense evaluation은 .. 뭘까요~~ 이 부분이 이해가 잘 안돼서 .. 추가 자료를 더 찾아 보고 업데이트 할 예정이다 ..
{:.notice}


## | Classification Experiments
### 01. Dataset
데이터 셋으로는 1000개의 클래스를 가지는 ILSVRC-2012 데이터 셋을 사용했다.

### 02. Single Scale Evaluation
#### Settings
Train scale S에 따른 Test scale Q의 설정은 아래의 표와 같다.

| Train scale S | Test scale Q |
| :-----------------: | :----------------: |
| 고정 값 | S, Train image scale과 같은 값을 가짐 |
| [Smin, Smax] 사이의 값 (scale jitter) | 0.5(Smin + Smax) |

#### Results

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/c4e1812e-8216-4679-9f61-85ae89c8acce){: .align-center}

1. Local Response Normalization(LRN) 사용 시 성능 저하가 나타났다.
2. 성능이 `B < B + 1x1 conv. (C) < B + 3x3 conv. (D)` 순으로 좋다.
- B < C : 비선형성 추가는 성능 향상에 도움이 된다.
- C < D : conv. filter를 통해 공간적 맥락을 파악하는 것이 중요하다.
3. Train image에 대한 scale jittering은 효과가 있다.

### 03. Multi Scale Evaluation
#### Settings
모델 평가에 Scale jittering을 적용해보는 실험이다.<br>
Train scale S에 따른 Test scale Q의 설정은 아래의 표와 같다.

| Train scale S | Test scale Q |
| :-----------------: | :----------------: |
| 고정 값 | S-32, S, S+32 |
| [Smin, Smax] 사이의 값 (scale jitter) | Smin, 0.5(Smin + Smax), Smax |

#### Results

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/1c3061f3-4ff1-4886-8469-ba8beaab6414){: .align-center}

학습과 마찬가지로 scale jittering을 적용하는 경우, 성능이 향상하는 모습을 보였다.

### 04. Multi Crop Evaluation
Dense ConvNet Evaluation과 Multi Crop Evaluation을 비교했다.

![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/4c09217d-6c88-4dfb-8f99-7e0ea0b72118){: .align-center}

1. 성능 : Multi crop evaluation > Dense ConvNet evaluation
2. Dense ConvNet Evaluation과 Multi Crop Evaluation은 상호 보완된다.

마지막으로 여러 모델의 결과를 평균내어 ConvNet Fusion을 만든 것에 대해 언급한다. 논문에 따르면, ILSRVC 제출을 통해 확인한 결과 두 Multi-scale 모델을 앙상블 한 경우 성능이 가장 좋았다고 한다.
{:.notice}

## | Conclusion
최대 19개의 레이어를 가지는 깊은 Convolutional Network를 평가했고, 이를 통해 깊이가 중요하다는 점을 확인했다.

## | 참고 및 출처
- VGGNet 논문
- Deep Learning Bible wikidocs : K_02. Understanding of VGG-16, VGG-19
 (https://wikidocs.net/164796)