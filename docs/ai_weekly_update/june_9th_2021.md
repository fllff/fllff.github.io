---
layout: default
title: 2021 June 9th
nav_order: -2
parent: AI Weekly Updates
---

## Measuring and Improving Consistency in Pretrained Language Models

### 개요
- [https://arxiv.org/pdf/2102.01017.pdf](https://arxiv.org/pdf/2102.01017.pdf)
- 38개의 relation에 대하여 328개의 paraphrase를 준비
- **동일한 의미의 문장들을 다른 어휘로 표현해 질의하면, PLM은 일관성을 유지 할 것인가**
    - 실험결과: not suitable for representing knowledge
- **일관성있는 모델을 위한 방법 제안**

### 내용
![image](https://user-images.githubusercontent.com/6601619/122449547-ecf47600-cfe0-11eb-8fa6-8072699a7ff0.png)
- knowledge base triplets로 paraphrase 문장들을 만들었다
- 동일 의미 문장에서, MASK에 대한 일관성을 확인해보니 별로였다
- **pretraining에 consistency loss를 적용하는 방법 제안**
    - **동일의미 문장 2개에 대한 softmax분포 결과 A, B 로 kl divergence를 구함**
    - kl은 비대칭 함수이므로, KL(AㅣB) + KL(BㅣA)를 사용
    - 이 결과를 MLM loss와 weight sum
    - BERT에서 일관성이 64.0% -> 80.8%로 개선

-----------------

## Implicit Representations of Meaning in Neural Language Models
### 개요
- [https://arxiv.org/pdf/2106.00737.pdf](https://arxiv.org/pdf/2106.00737.pdf)
- **Language Model은, 단순히 단어 등장 빈도를 통계낸 것인가? VS 세상을 표현하는 것인가?**
- text only 학습만으로도, context에 따른 **의미표현과 물체의 상태 암시 등이 일부 가능함**을 보임

### 내용
- representation과 label이 주어졌을 때, 작은 용량의 probe 모델(ex: linear classifier)로 representation에 대한 label을 학습한다.
- 상황을 변화시킨 후의 representation에 대한 probe 모델의 결과 변화를 확인한다.
![image](https://user-images.githubusercontent.com/6601619/122680677-5bc50f80-d22b-11eb-95c7-63d1458ee342.png)


-----------------

## Few-Shot Question Answering by Pretraining Span Selection
### 개요
- [https://arxiv.org/pdf/2101.00438.pdf](https://arxiv.org/pdf/2101.00438.pdf)
- **span selection을 활용한 QA용 pretraining 방법**
- 적은 수의 QA 데이터로 finetune 후 성능 비교했을 때 RoBERTa, SpanBERT 등 보다 월등히 좋음 
    - 72.7 F1 on SQuAD with only 128 training examples

### 내용
![image](https://user-images.githubusercontent.com/6601619/122680888-8499d480-d22c-11eb-9672-8928e2cfe75b.png)
- 되풀이 되는 span들 중 하나만 빼고 [QUESTION] 토큰으로 masking.
- 각 [QUESTION] 토큰의 representation 마다 답에 해당하는(마스킹 안된) 부분을 찾도록 pretraining 한다.
    - start position, end position 맞추기  
- fine-tuning시에는 "text[SEP]question[QUESTION]" 처럼 [QUESTION] 토큰을 맨 뒤에 붙여서 입력함
    - pretraining 시 학습된 information capture 능력을 활용하도록 유도

-----------------

## Zero-shot Fact Verification by Claim Generation

### 개요
- [https://arxiv.org/pdf/2105.14682.pdf](https://arxiv.org/pdf/2105.14682.pdf)
- **fact verification을 위한 데이터 생성 방법 제안**
    - wikipedia를 바탕으로 supported, refuted, unverifiable 태깅
    - zero-shot learning, few-shot learning 모두에서 도움이 되었음

### 내용
- 두가지 모델로 구성
    - Question Generation from evidence
        - 주어진 evidence와 text span에 대하여 text span을 Answer로 하는 question을 생성 
        - BART pretrain 후, SQuAD dataset으로 finetuning
            - 질문의 자연스러움에 대하여 여러 지표 + human evaluation 검증을 하였음
    - QA to Claim convertion
        - 주어진 QA로 claim을 생성
        - BART pretrain 후, QA2D dataset으로 finetuning
- Suppoerted 데이터 생성
    - evidence에서 NER로 entity 추출
    - 각 entity 마다 Question Generation
    - QA-to-Claim으로 supported claim 생성
- Refuted 데이터 생성
    - Question Generation 후, answer를 다른 entity로 교체
        - Sense2Vec으로 answer와 가장 비슷한 phrase들을 뽑은 후 하나를 선택 (word2vec의 일종)
        - 교체된 entity도 답일 경우를 대비해서, lexical overlap 적은 entity가 선택되도록 안전장치를 둠
        - 그럼에도 답일 가능성이 있는데, 샘플링해보니 2% 정도라 무시할만 하다고 함
    - QA-to-Claim으로 Refuted claim 생성
- Not Enough Info 데이터 생성
    - evidence가 있던 wiki 원문에서, evidence에 없는 문장을 랜덤으로 가져옴
    - evidence와 새로 가져온 문장을 expand
    - expand 된 문장 속 entity에 대하여 question과 claim을 생성
    - 이렇게 만든 claim은 기존 evidence와 관련되었지만, 기존 evidence 만으로는 unverifiable

-----------------

## An Attention Free Transformer

### 개요
- [https://arxiv.org/pdf/2105.14103.pdf](https://arxiv.org/pdf/2105.14103.pdf)
- self-attention (transformer)은 attention을 위해 dot product가 있고, 이것 때문에 n^2로 연산량이 많음
    - LHS를 사용하는 reformer 등등.. 의 방법들이 제안되어옴
    - (seq-len이 긴 이미지 쪽에서 특히 치명적)

### 내용
- **dot product를 element wise로** 대체하여 transformer를 구현하는 방식을 제안

-----------------

## Extrapolating to Unnatural Language Processing with GPT-3's In-context Learning: The Good, the Bad, and the Mysterious

(외삽법 (Extrapolation) 또는 보외법이란 이전의 실험으로부터 얻은 데이터들에 비추어, 아직 경험/실험하지 못한 경우를 예측해보는 기법이다. )

### 개요
- [https://ai.stanford.edu/blog/in-context-learning/](https://ai.stanford.edu/blog/in-context-learning/)
- **GPT-3 in-context learning의 특성 확인**
- few-shot으로 부자연스러운 규칙을 알려주고 잘 따르나 확인

### 내용
- unnatural date formatting
    - ![image](https://user-images.githubusercontent.com/6601619/122793960-b2e3e680-d2f6-11eb-9ae2-a1f9870f3c48.png)
    - 일반적이지 않은(unnatural) date formatting을 수행하도록 요청
    - 파싱하고, 섞는 작업이 가능한지 확인
    - 99.9% accuracy (1998/2000)
    - 일부 erorr가 왜 발생했는지 확인하기 위해, 여러 포맷을 시도해본 결과,
        - error들은 대부분 date-month 순서가 바뀐것들이었다 : pretrain때 자주 접한 date format을 따라간 경향
            - date, month, year 대신 의미없는 숫자들로 테스트해보니 swap으로 인한 오류는 찾기 힘듦
        - 최근 년도일수록 error가 많음 (데이터 부족 때문으로 추측된다고 함)
        
- unnatural classification 
    - ![image](https://user-images.githubusercontent.com/6601619/122796495-6cdc5200-d2f9-11eb-8003-d8b09c955cba.png)
    - ![image](https://user-images.githubusercontent.com/6601619/122796538-7a91d780-d2f9-11eb-9a9b-a9b4d6ecbb15.png)
    - pretrain 된 특성을 override 할 수 있는가
    - 골프, 축구, 하키 등을 동물로. 말, 오리 등을 채소로. 양파 옥수수 등을 운동으로 예측하도록 요청
    - 92.7% accuracy

- unnatural operator
    - 뺄샘 기호에 대하여 덧샘을 하도록 유도
    - in-context example을 많이 줄수록 accuracy 개선

### 결론
- good
    - 학습시 주어지지 않았던 unnatural input에 대해서도 잘 동작함
    - in-context example이 많을수록, 모델이 클수록 잘됨
- bad
    - accuracy가 높은것에 비해, 체계적이지 않아보임 (systematic 하지 않아보임 )
    - 틀리는 케이스들에 연관성을 찾기 힘듦
- mysterious
    - 학습된 일반적인 의미를 override 할 수 있음    
    - in-context example이 많을수록 점진적으로 override됨 
    - 이번 실험에서는, 일부 구간에 spike가 있었음

