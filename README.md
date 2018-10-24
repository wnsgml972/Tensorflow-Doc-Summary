# Tensorflow-Doc-Summary


## 분석 동기

블로그를 운영하는 블로거로서 만약 블로그를 자동으로 요약해서 글 제일 위에 나타낼 수 있다면
읽으러 오는 사람에게 얼마나 좋을까? 생각에 문서를 요약하는 프로그램을 생각하게 되었습니다.<br/>

먼저 영어 문서의 한에서 tensorflow 를 이용하여 문장을 요약한 것을 학습 시켜주는 프로그램을
만들었습니다. 하지만 영어 문서의 한에서는 한글 블로그에 문서 요약을 넣을 수 없으므로
TextRank 를 이용하여 한글 문서에서도 문서 요약을 가능하게 해주는 문서요약시스템을 간단히
구현해보았습니다.

* [Korea, TextRank-Doc-Summary](https://github.com/wnsgml972/TextRank-Doc-Summary)
* [English, Tensorflow-Doc-Summary](https://github.com/wnsgml972/Tensorflow-Doc-Summary)

####  프로젝트 바로가기 [:heavy_check_mark:](https://github.com/wnsgml972/Tensorflow-Doc-Summary/blob/master/tensorflow-doc-summary.ipynb)

<br/><br/>

## 개념 설명

텍스트 요약은 하나의 문장이 다른 문장에서 유추될 수 있는지를 식별하려는 논리 연습입니다.
텍스트 요약의 결과는 세가지 범주로 나타냅니다.

![개념](/img/1.png)

### 긍정의 요약

``"positive entailment(긍정적 요약)"``이라고 불리는 첫 번째 범주는 첫 번째 문장을 사용하여 두
번째 문장이 "참"임을 증명할 수 있을 때 발생합니다. <br/>
예를 들어 ``"Maurita and Jade both were at the scene of the car crash"`` ->
``"Multiple people saw the accident”`` 이 글을 보면 2 명이 사고 난 것을 잘 요약했다고 볼 수
있습니다. 그러므로 긍정적인 요약의 범주에 들어갑니다. <br/><br/>

### 부정의 요약

두 번째 범주인 ``"negative entailment(부정적 요약)"``는 긍정적 요약의 역입니다. 첫 번째 문장을
사용하여 두 번째 문장을 반증 할 수 있는 경우에 발생합니다. <br/>
예를 들어 ``"Two dogs played in the park with the old man"`` -> ``"There was only one canine in
the park that day"`` 다음과 같은 문장의 2 마리 개에 대한 정보가 없어졌으므로 올바른 요약이
아닙니다. 그러므로 부정적인 요약의 범주에 들어갑니다. <br/><br/>

### 중립의 요약

마지막으로 두 문장이 상관 관계가 없다면 ``"중립적 요약(universal entailment)"``이 있는 것으로
간주됩니다. <br/>
예를 들어 ``"I played baseball with the kids"`` -> ``"The kids love ice cream"`` 두 문장은 서로
상관관계가 없으므로 중립적 요약의 범주에 들어갑니다.

<br/>

## Step 1. 필요한 패키지 설치

* ``pip install tensorflow``
* ``pip install numpy``
* ``pip install matplotlib``
* ``pip install six``
* ``pip install tqdm``

<br/>

## Step 2. 데이터 수집

스탠포드의 Glove 데이터를 사용하였습니다. Glove 데이터는 벡터화를 위해 미리 훈련된 데이터
입니다.

![2](/img/2.png)

텍스트 요약을 위한 dev, test, train 데이터는 스탠포드의 SNLI 데이터를 이용하였습니다.

![2](/img/3.png)

<br/>

자료의 수집 방법은 six 라이브러리를 이용해 zip 파일을 다운받고

![2](/img/4.png)

unzip 하여 text 파일을 추출하는 방법을 이용하였습니다.

![2](/img/5.png)

<br/>

## Step 3. 문장에서 딥 러닝을 하기위해 입력 모델 만들기

1. __glove 벡터 파일을 이용해 공백으로 구분된 사전을 만든 후 토큰화합니다__

![3](/img/6.png)

2. __시퀀스를 만들어주는 함수를 정의합니다.__

![3](/img/7.png)

3. __기본 RNN은 복잡해지면 학습에 어려우므로 LSTM을 이용해 학습합니다.

![3](/img/8.png)

이전에 추가 한 기본 RNN 셀을 포함하지 않도록 그래프를 다시 설정 시켜준 후 다시
BasicLSTMCell을 불러 LSTM을 정의해야 합니다.

![3](/img/9.png)

4. __정규화를 위해 dropout을 구현합니다.__

LSTM 레이어만 사용했으므로 “a”, “an” 같은 부정적인 결과를 낼 수 있는 좋지 않은 여러 문장이
있습니다. 이러한 문장은 하나로 통일되야 합니다. 그러므로 __정규화를 해야합니다.__ <br/><br/>

본 프로젝트에서는 신경망의 크기가 커짐에 따라 최종 결과를 계산하는데 사용되는 파라미터의
수가 증가되고 각 파라미터는 한꺼번에 교육을 받게 되어 overfitting이 발생합니다. <br/><br/>

따라서 Dropout을 이용해 몇 개의 노드를 죽이고 남은 노드들을 이용해 훈련해야 합니다. 본 프
로젝트에서는 tensorflow의 DropoutWrapper를 이용했습니다.

![3](/img/10.png)

5. __모델 완성__

양방향 RNN을 사용하여 입력 데이터를 통해 앞뒤로 실행시켜 가설과 증거를 검토합니다. <br/>
LSTM의 최종 출력물인 분류 점수를 얻어냅니다. <br/>
분류 점수는 텍스트 요약에 대해 결과에 대한 신뢰도를 결정할 수 있습니다.

![3](/img/11.png)

<br/>

## Step 4. 완성된 모델을 이용해 학습시키기

1. __훈련에 앞서 정확도를 나타내는 accuracy와 현재의 가중치에서 틀린 정도를 알려주는
loss 함수를 정의합니다.__

![4](/img/12.png)

기존의 GD 방식을 이용해 cost를 측정하는 방식은 시간이 너무 많이 소요되어 전체를 살펴
보고 계산하는 방식이 아닌 조금씩 나눠서 빠르게 일부 데이터만 계산하는 SGD 방식을 이
용하였습니다.

2. __학습시킵니다.__

tqdm을 이용해 학습 진행 사항을 추적하도록 만들었습니다.

![4](/img/13.png)

학습 과정입니다.

![4](/img/14.png)

높은 정답률이 나온 것을 확인할 수 있습니다. 처음에는 snli_dev_file을 이용해 테스트하여 Loss가
0.7~0.5로 유지되어 조금 더 loss를 줄이기 위해 훨신 더 큰 파일인 snli_full_dataset_file을 이용해
학습하였습니다. loss가 엄청나게 준 것을 확인할 수 있습니다. <br/><br/>

너무 높은 정답률이 나와 여러 방법을 시도해보았지만 방법을 찾지 못 하였습니다. ㅠㅠㅠㅠ

<br/>

## Step 5. 검증하기

적절한 증거 문장과 가설 문장을 삽입하여 검증합니다. 밑의 그림에서는 사고가 나는 상황을 적
절히 표현했으므로 Positive가 정답입니다. Label과 Prediction이 모두 Positive로 정답을 맞춘 것을
확인할 수 있습니다.

![4](/img/15.png)
