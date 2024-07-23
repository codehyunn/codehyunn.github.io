---
title: "[논문 리뷰] LoRA: Low Rank Adaptation of Large Language Models"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Paper-Review
tags:
  - Paper
permalink: /paper-review/06/
toc: true
toc_sticky : true
last_modified_at: 2024-07-23
---

이번 리뷰는 논문 구조를 그대로 따르지 않고 내용을 간추리고 재구성하여 정리했다.<br>
논문 링크는 [여기](https://arxiv.org/pdf/2106.09685)!


**🧚🏻‍♀️짧게 정리해보자면 . . .**<br>
LoRA는 low-rank matrices를 활용하여 효율적인 학습을 가능하게 한 적응(adaptation) 방법이다.
{:.notice--blog-theme}


## | Background

자연어 처리에서는 한번 학습된 큰 모델을 적응(adapting)하여 특정 테스크(downstream task)에 적용한다. 이 과정에서 주로 사용되는 방법은 fine-tuning이다. Fine-tuning은 모델의 모든 파라미터를 업데이트 하는 방법이고, 모델이 커질수록 매우 비효율적이라는 문제점을 가진다. 이를 보완하기 위해 adapter layers, prefix tuning과 같은 방법이 제안되었으나, 대부분 추론 시간이 지연(Inference latency)되거나 성능이 떨어지는 모습을 보였다.

**🐨 Prefix Tuning**<br>
프롬프트를 잘 작성하면 좋은 성능을 얻을 수 있다는 생각에서 고안된 방법이다. 입력 앞에 Prefix를 붙이고, 태스크 별로 적당한 Prefix를 학습한다. 학습 과정 동안 모델의 파라미터가 고정됨에도 불구하고, 상황에 맞는 단어를 생성할 수 있다는 장점을 가진다. 자세한 내용은 아래 블로그 참고하시기를!<br>
🔗 참고한 블로그 : [ffighting](https://ffighting.net/deep-learning-paper-review/language-model/prefix-tuning/)
{:.notice}    

저자들은 **"Change in weights during model adaptation has a low intrinsic rank'**라는 가설을 바탕으로 **Low-Rank Adaptation(LoRA)**을 제안했다. LoRA는 dense layer의 rank decomposition matrices를 최적화하여 일부 dense layer를 간접적으로 학습하는 방법이다. Fine-tuning과 달리 모든 파라미터를 업데이트 할 필요가 없기 때문에 시간이나 비용을 줄일 수 있다는 장점을 가진다.

**🐨 가설 관련**<br>
논문에서 "Learned over-parametrized models in fact reside on a low intrinsic dimension"라는 점에서 영감을 얻었다고 언급했다. (관련 논문은 [여기](https://arxiv.org/abs/2012.13255))
{:.notice}    


## | Related Works

네 가지의 관련 분야를 소개해주었는데, 주제만 언급하고 넘어가겠다.

- Transformer Language Models
- Prompt Engineering and Fine-tuning
- Parameter-Efficient Adaption
- Low-Rank Structures in Deep Learning


## | Methods
### Problem Statement
이 논문에선 언어 모델링에만 집중했다고 하며, 문제 상황은 아래와 같이 정의한다.

- Pre-trained autoregressive language model $$P_{\Phi}(y\vert x)$$이 주어졌으며, 이 모델은 $$\Phi$$를 파라미터로 가지는 트랜스포머 모델이다.
- 타겟 테스크는 conditional text generation이다.

적응 방법(adaptation method)에 따른 학습 과정은 다음과 같다.

- Full Fine-tuning
    - 모델은 사전 학습된 가중치인 $$\Phi_0$$로 초기화하고, 학습을 진행하면서 $$\Phi_0 + \Delta\Phi$$로 업데이트한다.
    - 테스크에 따라 모델과 같은 크기의 파라미터를 모두 학습해야 한다. ($$\vert\Phi_0\vert = \vert\Delta\Phi\vert$$)
    - 목적 함수는 아래와 같다.

    $$\max\limits_\Phi \sum\limits_{(x,y)\in Z} \sum\limits_{t=1}^{\vert y \vert} log (P_\Phi (y_t\vert x,y_{<t}))$$

- RoLA (Row Lank Adaptation)
    - 위와 마찬가지로 $$\Phi_0$$로 초기화하고, 학습을 통해 $$\Phi_0 + \Delta\Phi$$로 업데이트 된다.
    - 하지만 아주 작은 파리미터 셋인 $$\theta$$에 대해 $${\Delta\Phi = \Delta\Phi(\theta)}$$가 성립하기 때문에 $$\theta$$에 따른 $$\Delta\Phi$$을 최적화하면 된다.
    ($$\vert \theta \vert \ll \vert \Phi_0 \vert$$)
    - 목적 함수는 아래와 같다.

    $$\max\limits_\theta \sum\limits_{(x,y)\in Z} \sum\limits_{t=1}^{\vert y \vert} log (P_{\theta_0+\Delta\Phi (\theta)} (y_t\vert x,y_{<t}))$$


다시 정리해보자면 fine-tuning은 모델의 모든 파라미터를 찾아야 하는 반면, RoLA는 훨씬 더 적은 파라미터에 대해 최적화하면 된다. 하나의 모델을 다양한 테스크에 적용한다고 가정했을 때, RoLA는 시간과 비용을 아낄 수 있는 효율적인 방안이 될 수 있다는 것을 짐작할 수 있다.

### Low Rank Parametrized Update Matrices
어떤 태스크에 대해 학습할 때, 사전 학습된 언어 모델은 낮은 내재적 차원(intrisic dimension)을 가지며, 더 낮은 차원의 공간에 투영되더라도 학습이 효과적으로 이루어진다고 한다. 이 사실을 바탕으로 가중치 변화도 낮은 내재적 Rank(intrisic rank)를 가진다고 가정한다.

그렇다면 이게 어떻게 학습에 적용되느냐. 식으로 살펴보면 간단하다. 사전 학습 된 가중치는 $$W_0$$ ($$\small{W_0 \in R^{d\times k}}$$), 업데이트 된 가중치는 $$W_0 + \Delta W$$라고 하자. 가정에 따라 가중치 변화를 낮은 rank $$r$$를 가지는 행렬의 곱으로 표현할 수 있다. $$\Delta W = BA$$ ($$\small{B\in R^{d\times r}}$$, $$\small{A \in R^{r\times k}}$$, $$\small{\ r \ll min(d,k)}$$). 이 식을 순전파 과정에 그대로 대입하면 $$h = W_0x + \Delta Wx = W_0x + BAx$$이 된다. 추가적으로, 학습이 이루어지는 동안 $$W_0$$은 고정되고 $$A,B$$만 업데이트 된다.

논문에 따르면 LoRA는 full fine-tuning의 일반화 버전으로 볼 수 있다. 학습 가능한 파라미터의 수를 증가시켰을때 adapter-based methods는 MLP나 prefix-based methods로 수렴되는 데에 반해 LoRA는 기존 모델을 학습하는 것과 유사해지기 때문이다. 대신 fine-tuned 된 모델과 달리 간단하고 추가적인 지연 없이 태스크를 변경할 수 있으며, 메모리 오버헤드도 줄어든다는 장점을 가진다.

**🐨 태스크 변경**<br>
태스크를 변경할 때에는 $$W_0 + B_1A_1$$ 에서 $$B_1A_1$$를 빼고 다른 $$B_2A_2$$를 더해주기만 하면 된다.<br>모델 전체를 다 학습해야 했던 full fine-tuning과 비교하면 훨씬 간단하면서도 빠른 방법이지 않는가 !! 
{:.notice}   

### Applying LoRA to Transformer
이론적으로 모든 가중치에 LoRA를 적용할 수 있지만, 논문에서는 적용 범위를 Transformer의 Attention weights로 제한하였으며 그 외의 부분은 고정하였다. 

실험을 통해 메모리와 저장공간을 아낄 수 있다는 점과 태스크를 변경할 때의 비용이 매우 크게 감소한다는 점을 알 수 있었다. 하지만 하나의 순전파 과정에서 서로 다른 태스크를 처리하기는 어렵다는 한계점도 존재했다.

### Benefits
지금까지 소개 된 장점들을 한번에 정리해보면 다음과 같다.

- 메모리와 저장공간 사용량을 줄일 수 있고, 태스크 변경 시 오버헤드도 줄어든다.
- 그래디언트 계산이 없기 때문에 효율적인 학습이 가능하다.
- 추론 지연(Inference latency)이 없다.
- 기존 방법들과 독립적이고, 결합하여 사용 가능하다.


## | Experiments and Results

### Settings
- task : NLU(Natural Language Understanding), NLG(Natural Language Generation)
- models : RoBERTa, DeBERTa, GPT-2, GPT-3

### Baselines
실험을 통해 비교할 방법들은 다음과 같다.

- **Finetuning** : 모델은 사전 학습된 가중치와 편향으로 초기화 하고, 그래디언트가 업데이트 되면 모든 파라미터가 업데이트 된다.
- **Bias only or BitFIt** : bias vector만 학습하고 나머지 부분은 고정하였다.
- **Prefix embedding tuning (PreEmbed)** : 특정한 토큰을 입력 토큰에 주입하는 방법이다. 이때 토큰은 모델의 사전에는 존재하지 않지만, 학습이 가능한 단어 임베딩을 가진다.
- **Prefix-layer tuning (PreLayer)** : Prefix-embedding tuning의 확장 버전으로, 특정 토큰에 대한 단어 임베딩이 아니라 transformer layer 뒤에 오는 활성화 함수를 학습한다.
- **Adapter tuning** : self attention module, MLP module, subsequent residual connection 사이에 adapter layer를 추가하는 방법이다.
- **LoRA** : 가중치 행렬에 학습 가능한 rank decomposition matrices 쌍을 추가하는 방법이다.

### Results
![image](https://github.com/user-attachments/assets/e96f4158-b6b8-42e8-bceb-358c5ebff19a){:.align-center}

위의 표는 GPT-2에 대한 실험 결과이다. 논문에서는 GPT-2 외에도 GPT-3, BERT 모델들의 실험 결과를 발표했다. 모든 결과를 살펴보면 전반적으로 LoRA가 다른 방법들에 비해 학습 가능한 파라미터 수가 적음에도 불구하고 좋은 성능을 낸다는 것을 알 수 있었다.

## | Analysis
Low-Rank Updates에 대한 추가적인 내용이다. 결론만 간략하게 정리했다.

### Which weight matrices in Transformer should we apply LoRA to?
- LoRA를 큰 Rank를 가지는 하나의 가중치에 적용하는 것보단 Rank가 낮은 여러 개의 가중치에 적용하는 것이 효과적이다.
    
### What is the Optimal Rank r for LoRA?
- 작은 r로도 좋은 성능을 보인다. 
- $$\Delta W$$ 는 아주 작은 내재적 rank를 가지며, r을 키운다고 해서 더 의미있는 공간을 커버하진 않는다. 

### How does the adaptation matrix $$\Delta W$$compare to W?
- $$\Delta W$$중 일부는 $$W$$의 피처를 구체화 할 정도로 높은 상관관계를 가진다.
- 이때 업데이트를 통해 구체화 된 피처는 기존 모델에서 강조되지 않은 피처이다.

## | Conclusion and Future works
### Conclusion
- LoRA (Low-Rank Adaptation) 제안했다.
- Inference latency, 입력 시퀀스 길이 감소 없이 모델의 성능은 높이는 효율적인 학습(adaptation) 방법이다.
- dense layer를 가지는 다른 신경망에 적용 할 수 있다.

### Future works
- LoRA와 다른 adaptation method의 결합
- 사전 학습된 피처들이 태스크에 맞게 바뀌는 과정
- LoRA를 적용할 가중치 행렬 선정 방법 등 ...

