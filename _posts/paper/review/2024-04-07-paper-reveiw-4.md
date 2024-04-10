---
title: "[논문 리뷰] Attention is all you need"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Paper-Review
tags:
  - Paper
  - Transformer
permalink: /paper-review/04/
toc: true
toc_sticky : true
last_modified_at: 2024-04-10
---

이번에 리뷰할 논문은 Transfomer를 발표한 "Attention is all you need"이다. 논문 링크는 [여기](https://arxiv.org/pdf/1706.03762.pdf) !

**🧚🏻‍♀️짧게 정리해보자면 . . .**<br>
Transformer는 RNN이나 CNN 없이 Attention만으로 구성된 최초의 모델이다.
{:.notice--blog-theme}


## | Introduction

### Recurrent Networks (RNN, LSTM, GRU)
- RNN 계열의 모델들이 Sequence modeling, *Transductive learning 분야에서 SOTA 성능을 보임.
- 이전 시점의 정보를 이용하여 다음 시점의 Hidden state vector를 계산하는 RNN 모델의 특성 상 병렬 학습이 불가능함. 
- 그 결과, 긴 시퀀스에 대해선 한계점이 존재함.
  - 해결하고자 하는 노력이 많았고, 성능이 오른 사례도 있었지만, 근본적인 문제 해결은 이루어지지 않음.

### Attention mechanisms
- 입출력 시퀀스 사이의 거리에 상관 없이 모델링이 가능하도록 함
- RNN 계열의 모델들과 함께 사용됨

### Transformer
- Attention mechanism만으로 구현된 모델인 **Transformer**을 제안
- 거리에 상관없이 입력과 출력 사이의 관계를 모두 고려할 수 있으며 병렬화가 가능함

**🐨 Transductive learning**<br>
관찰된 데이터(▶️학습 데이터)를 통해 특정 데이터(▶️검증 데이터)를 예측하는 태스크를 의미한다.
{:.notice}

## | Background
### Convolution Neural Network based Models
- 연산량을 줄이기 위해 Convolution Neural Network(CNN)을 기반으로 하고, 병렬적으로 hidden representation을 계산하는 모델이 등장함. 
  - ex. Extended Neural GPU, ByteNet, ConvS2S
- 입출력 간의 거리에 따라 계산량이 크게 달라지는 모습을 보임.


### Self-attention & End-to-end memory network
- Self-attention (intra-attention)
  - 각 position의 Representation을 구하기 위해 다른 position의 시퀀스를 연결하는 Attention mechanism
- End-to-end memory netrwork
  - 반복적인 Attention machanism을 가지는 구조

### Transformer
- RNN이나 CNN 없이 "Only self-attention" 만으로 입출력의 representation을 계산한 최초의 모델
- CNN 기반의 모델들과 달리 연산량이 일정함

## | Architecture
### Overall architecture
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/2e922830-248f-4733-a6ea-b91c833afce9){:.align-center}

- Neural sequence transduction model과 비슷한 구조를 가짐
  - Input($$x_1, \cdots, x_n$$) → ENCODER → Representation($$z_1, \cdots, z_n$$) → DECODER → Output($$y_1, \cdots, y_n$$)
- Encoder와 Decoder가 stacked self-attention, point-wise fully connected layer로 구성됨

### Encoder and Decoder
- Encoder와 Decoder 모두 6개의 layer로 이루어짐 (N=6)
- Encoder는 2개의 sub-layer, Decoder는 3개의 sub-layer로 구성되어있음
  1. Encoder
    - Multi-head self attention mechanism
    - Position-wise fully connected feed-forward network
  2. Decoder
    - Multi-head self attention mechanism<br>
      → Decoder에서는 이어지는 정보를 참고하지 않게 하기 위하여 변형된(=Masked) 구조를 사용함
    - Position-wise fully connected feed-forward network
    - Multi-head self attention mechanism over Encoder outputs

- 모든 sub-layer에는 Residual connection과 Layer noramlization이 적용됨
  - Residual connection을 수행하기 위하여, 모든 레이어의 Output은 동일한 차원($$d_model$$,512)을 가짐

### Attention
#### Scaled Dot-Product Attention
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/cfd3883d-0beb-47e8-9fe6-ff858f3538f9){:.align-center}
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/7a744e63-ae8c-4c60-bb76-a78609e8f55a){:.align-center}

- Q와 K는 $$d_k$$차원의 query와 key vector로 이루어진 행렬이고, V는 $$d_v$$차원의 value vector로 이루어진 행렬임
- 일반적으로 가장 많이 사용되는 attention function은 Additive attention과 Dot-product attention임
  1. Additive attention
    - 하나의 hidden layer를 가지는 Feed-forward network로 attention을 계산하는 방법
  2. Dot-Product attention
    - Scaled Dot-Product attention에서 scaling factor를 제외한 것과 같음
    - Additive attention 보다 빠르고 효율적임
- $$\sqrt{d_k}$$로 Scaling하는 이유
  - $$d_k$$가 커질수록 dot product의 규모가 커짐
    - $$q,k$$의 mean=0, variance=1일 때, $$q\cdot k$$은 mean=0, variance=$$d_k$$를 가짐.
  - 이어서 Softmax의 범위도 커지고, Gradient도 매우 작아지게 됨
  - 이를 방지하기 위해 Scaling 진행.

#### Multi-Head Attention
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/154af730-21ae-4f5c-b67e-3997b00f44e8){:.align-center}
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/daddbc3e-994f-4e0b-81be-926ac6b3b1bd){:.align-center}

1. key, value, query를 각각 h번 선형 변환
  - 이때 가중치(parameter matrix)는 서로 다름
2. attention function을 병렬적으로 적용 
3. h개의 결과를 concat하고 마지막으로 선형 변환

#### Applications of Attention
1. Encoder-Decoder attention
  - Query는 previous decoder layer에서, Key와 Value는 encoder output에서 옴
  - Decoder의 모든 position에서 입력 시퀀스의 모든 position를 참고할 수 있음
2. Encoder의 Self-attention 
  - Query, Key, Value 모두 previous encoder layer의 output임
  - Encoder의 모든 position에서 previous encoder layer의 모든 position를 참고할 수 있음
3. Decoder의 Self-attention
  - Query, Key, Value 모두 previous decoder layer의 output임
  - Decoder의 모든 position에서 현재 position까지 참고할 수 있음
  - *auto-regressive property를 지키기 위하여 Decoder에서 *leftward information flow를 막음
  - softmax를 적용하기 전에 미래 시점에 해당하는 값들을 $$-\inf$$로 마스킹함

**🐨 Auto-regressive & Leftward information flow**<br>
▶️Auto-regressive : 시계열 데이터에서 현재 값이 이전의 값들에 의존하는 특성<br>
▶️Leftward information flow : 현재 시점보다 미래 시점의 정보를 보고 예측하는 것을 의미
{:.notice}

### Position-wise Feed-Forward Networks
- 모든 Attention sub-layer 다음에는 Fully connected feed-forward network(FFN)가 있음
- 모든 FFN의 구조는 같지만, 파라미터는 다름
- 두 번의 선형 변형과 그 사이의 ReLU 함수로 이루어졌으며 입출력 차원은 $$d_{model}$$(512)임.
![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/d96b79ce-2141-4cd4-91b7-065517150e8d){:.align-center}

### Embeddings and Softmax
- 입출력을 $$d_{model}$$(512)차원의 벡터로 변형시키기 위해 Learned embeddings 사용
- 출력을 다음 토큰의 예측 확률 값으로 변형시키기 위해 Learned linear transformation, softmax function 사용
- 여기서 언급된 Embedding layer와 linear transformation은 같은 weight를 사용함

### Positional Encoding
- Attention 구조는 위치에 대한 정보를 반영하지 않으므로 절대/상대적인 위치 정보를 따로 주입함
- 입력 embedding에 Positional encodings를 더하는 방식으로, positional encoding도 $$d_{model}$$(512)의 차원을 가짐
- 논문에서는 sine, cosine 함수를 이용하여 positional encoding을 구함
  ![image](https://github.com/codehyunn/codehyunn.github.io/assets/87523224/e5d14917-be2f-49c6-9d08-d57718a3ca5c){:.align-center}
  - $$pos$$는 position을, $$i$$는 차원을 의미함
  - 각 차원의 positional encoding은 sinusoid의 형태를 띄는데, 차원마다 다른 주기를 가지는 함수에 매칭됨
  - sine, cosine 함수를 통해 상대적인 위치를 참고하는 것이 선형 함수를 이용하여 위치를 파악하는 것보다 학습에 도움이 될 것이라고 예상하여 sine 함수를 선택함
  - 실험 결과, Learned positional embedding과 비슷한 성능을 보임. 그러나 sinusoid 버전은 모델이 학습한 데이터보다 더 긴 시퀀스를 만들어 낼 수 있었기 때문에 최종적으로 이를 선택함

## | Conclusion
- Transformer는 attention만으로 이루어진 first sequence transduction 모델임
- multi-headed self-attention을 통해 encoder-decoder 구조의 recurrent 레이어를 대체함
- 기계 번역 분야에서 다른 모델들에 비해 빠른 학습 속도와 높은 정확도를 보임
