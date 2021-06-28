---
layout: default
title: 2021 June 2nd
nav_order: -1
parent: AI Weekly Updates
---

## Drawing Multiple Augmentation Samples Per Image During Training Efficiently Decreases Test Error

### 개요
- [https://arxiv.org/pdf/2105.13343.pdf](https://arxiv.org/pdf/2105.13343.pdf)
- 컴퓨터 비전에서 mini-batch의 이미지마다 증강된 데이터를 하나씩 생성하는것이 표준처럼 사용되지만, generalization을 위한 최적의 방식인지는 확인되지 않았다.
- SGD의 subsampling을 통해 gradient를 계산함에 따라 생기는 gradient variance는 정규화의 효과가 있긴 하지만,
- data augmentation으로 인해 생기는 variance는 test accuracy에 방해가 된다.

### 내용
- **benefit of data aug arise from bias, not variance**
    - (figure 중에. 이미지당 증강 횟수가 늘어남에 따라, 동일 epoch에서 더 낮은 train cross entropy를 가짐을 보여주는 그래프가 있었는데, 이것만으로도 empirical 주장이 가능하다고 생각함) 
- **이미지당 더 많은 증강이미지를 생성한 경우, minibatch안에 unique 이미지가 더 적어도 test accuracy가 좋았다.**
    - (unique 이미지가 적어서, gradient variance가 커지고, 결국 성능이 낮아져야 함에도 불구하고...) 
- batch size가 큰 경우(1024), 증강 이미지 수와 상관 없이 비슷한 lr에서 최고의 성능을 보였지만,
- batch size가 작은 경우(64), 증강 이미지 수가 많을수록 최고의 성능을 보이는 lr이 작았다.
- SGD의 정규화 효과는 minibatching으로 인한 variance로 부터 오는것 이므로, augmentation으로 인한 일반화 효과는 lr과 batch 안의 unique 이미지 수와 관계가 있다고 예상 할 수 있다.
    - (최적의 lr은 batch size가 중요한 것이 아니라 minibatch 속의 unique 이미지 수에 따른 것이라는 뜻 같다.)
    
-----------------

## Medically Aware GPT-3 as a Data Generator for Medical Dialogue Summarization

### 개요 
- [https://www.aclweb.org/anthology/2021.nlpmc-1.pdf#page=76](https://www.aclweb.org/anthology/2021.nlpmc-1.pdf#page=76)
- medical diaglogue summarization에서는, 모든 의학 관련 정보를 놓쳐서는 안됨
- 하지만 summurization model 학습을 위해선 많은 labeled data가 필요하고 얻기 힘듦. 
- GPT-3를 활용하여 학습용 데이터를 생성할 수 있음

### 내용
- GPT-3를 그냥 사용할 경우, 중요한 의학 정보들이 생략됨
- **GPT-3에 의학 정보를 주입하기 위해, GPT-3-ENS를 고안함**
```
0. label이 없는 데이터(D라고 하겠음)를 준비
1. labeled dataset에서 일부 샘플링 (비복원 추출)
2. 샘플링된 데이터로 few-shot learning 하여 D에 대하여 summary 생성
3. 1~2를 K번 반복하여 K개의 summary 확보
4. Medical Entity Recognizer를 통해, K개의 summary 각각 에서 medical entity를 추출.
5. D에서도 medical entity 추출
6. medical entity의 recall이 가장 높은 summary 정답 label로 한다.
```
- 실제로 GPT-3-ENS를 모델로 사용하면 개인 정보가 API로 전송되어야하는 위험성이 있기 때문에, data labeler로만 사용했다
    - summarization model은 PEGASUS와 DRSUM 사용
- 의사들을 대상으로한 평가에서 GPT-3-ENS로 확보한 데이터와 사람이 태깅한 데이터를 같이 사용하여 좋은 결과를 보였다

-----------------

## ByT5
- **works on raw utf-8 bytes (no tokenizer)**
- tokenizer의 문제점
    - finite vocab (=unseen 문제 발생 가능)
    - 오타 등의  노이즈에 약함
    - 엄밀히 말하면 end to end가 아님
    - multilingual 일 경우, vocab matrix가 너무 큼
- byte level 사용하면 위의 문제들을 해결하지만, seq len이 길어지는 단점이 있음
- 기존에 나온 byte-level 모델들은 seq len이 길어져서 computational cost가 많아짐에 집중.
    - ex) convolutional front layer, downsampling... 
- ByT5는 token-to-token 모델인 mT5를 byte level로 바꾸기 위해,
    - bigger encoder, smaller decoder 사용
    - longer span mask during MLM pretrain
=> 일반적인 text 에서는 token level과 비슷한 성능을, noisy 상황에서는 dramastically better 성능을 보였다고 한다. 

-----------------

## Knowledge Inheritance for pre-trained language models

### 개요
- [https://arxiv.org/pdf/2105.13880.pdf](https://arxiv.org/pdf/2105.13880.pdf)
- **기존의 잘 학습된 PLM을 활용하여 larger PLM을 학습 할 수 있을까**
    - Knowledge Inheritance (KI)라는 방법을 제안
    - self-learning과 teacher-guided learning을 통합

### 내용
- 용어
    - 기존의 작은 PLM: M-s 
    - 신규 학습하려는 큰 PLM: M-l
- Knowledge Inheritance
    - M-s, M-l 사이의 kl divergence와 M-l의 학습 loss를 합쳐서 사용 
- Dynamic Inheritance Rate
    - M-l은 학습됨에 따라 M-s보다 뛰어나게 될 것이므로, 학습이 진행될수록, teacher의 지식을 점점 적게 받아들이도록 함
- Diverse Teachers & Domains
    - 각 도메인의 corpus로 학습된 여러 M-s들을 모두 활용하는 방법 
    - corpus soucre 마다 optimal teacher를 선택하고, 해당 corpus 학습시에는 선택된 optimal teacher로 학습

