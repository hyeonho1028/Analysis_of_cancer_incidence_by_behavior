# Analysis of cancer incidence by behavior


## 1 데이터 이해
### 1. codebook
1. 컬럼에 대한 설명
2. 건강관련 위험 행동
3. 만성 건강 상태 및 예방 서비스 사용
4. 미국 거주자에 대한 주 데이터를 수집하는 건강 관련 전화 조사 시스템, 설문조사 시스템
5. 11년 ~ 16년 사이의 Data가 존재
6. 암은 흡연, 음주 등의 다양한 요인에 의한 영향으로 발병하는 복합적인 행동적 과정을 거치는데, 이 과정을 해석해 보고자 한다.

### 2. EDA와 시각화를 통한 이해
1. R의 ggplot2 패키지를 사용하여 visualization을 하였다.
2. 범주형 데이터가 주를 이뤘고, 연속형 데이터의 경우 6개의 컬럼이 있었다.
3. y가 0과 1의 비율이 큰 차이가 났다.(1의 비율이 암이 걸린 사람인데 매우 낮은 것을 보임)

## 2 데이터 전처리
### 1. missing value
1. 95%이상의 데이터를 가지고 있는 컬럼은 제거 하였다.
2. 상관관계가 너무 높아 다중공선성을 유발하거나, 해석의 여지를 낮추는 컬럼도 제거하였다.

### 2. outlier
1. 응답을 거부하거나, 설문을 아예 시도 하지 않은 인원들에 대한 outlier가 존재하는데 이 때, 그 자체로 의미가 있다고 판단하여 동일 범주로 묶어주었다.
2. 동질성 검정과 독립성 검정을 시도하여 이상값에 대해 한번 더 검증을 하였다.(범주형 변수만 시도)
3. 차원이 너무 크기 때문에, 차원축소가 필요함을 느꼈다. 
4. 차원축소의 논문을 여러개 찾아보니, 불균형 데이터에 대해 랜덤포레스트를 이용한 변수선택방법이 bagging, bumping을 이용한 변수선택법보다 개선된 점들을 볼 수 있었다. 물론 일반화는 위험하므로, 추가적인 방법론을 사용하여 모델을 사용하여 변수를 선택하였다.
5. 이런 방법은 해석이 불가하다는 점과 속도가 느린 단점이 있으나, 높은 예측력을 가진 모델을 개발할 수 있으므로 목적에 따라 변수 선택 및 모델을 선택하면 된다고 판단했다.

## 3 데이터 모델링
### 1. 사용한 모델
1. logistic regression : 로지스틱 회귀모형은 목표변수가 두 집단 또는 그 이상의 집단으로 나누어져 있는 경우에 개별 관측치들을 분류하고 예측하는 데 사용되는 대표적인 분류기법이다. 이 모형의 특징은 선형회귀모형과 유사하여 회귀계수와 오즈비를 이용한 해석이 용이하다는 점과 필요하지 않은 변수는 제거하고 꼭 필요한 변수만을 선택함으로서 모형의 예측력과 해석력을 높일 수 있다.
2. C4.5 : 이것은 1993년 Quinlan에 의해 제안된 방법으로써 기본적인 개념은 트리기반의 분류 알고리즘인 ID3와 동일하지만, ID3의 단점인 연속형 속성처리, 결측치 처리 등이 보완된 방법이다. CART(이진분류만을 지원함)와는 다르게 가지의 수를 다양화 할 수 있다. 연속형 변수에 대해서는 2개 범주로 이진분리를 하지만, 범주형의 경우 변수가 갖는 범주 개수만큼 분리를 수행한다. 이것의 단점은 지나치게 단순화되는 상황으로 문제가 될 수 있다. 분리기준은 information gain ratio(정보이익비율)이며, 감소되는 엔트로피 양을 뜻한다. 개선이 되지 않을 때까지 나무가 형성되기 때문에 불순도가 0이 될때까지 가지를 치므로, 오버피팅의 위험이 높다. 그렇기 때문에, 적당한 크기의 나무를 만들기 위해 pruning을 실시하여 오버피팅을 방지한다.(elbow)
3. randomforest : 랜덤포레스트의 목적은 의사결정나무 모형을 다수 만들어 더 정확한 예측을 하는 것이다. 랜덤포레스트의 특징은 모든 변수를 사용하는 의사결정나무와는 다르게 설명변수를 무작위로 선택하고, 선택된 설명변수의 집합 중에서 가장 최적의 결과를 내는 방법을 이용한다. 또한, 복원추출을 실행하여 n개의 표본을 추출하고 (범주의 경우 m=M^1/2, 연속의 경우 m=M/3)개의 변수를 랜덤으로 선택하여 적용했다. 이렇게 선택된 변수들로 의사결정나무를 만들어 최적의 split 조합을 찾아 각각의 N개의 단일 분류자 집합을 얻고, bagging 방법론과 마찬가지로, 반응변수가 연속형인 경우 평균, 버뭊형인 경우 투표를 사용하여 단일 분류자를 결합한다. 랜덤포레스트의 단점은 해석이 어렵다는 것이 있으나, 중요도지수, 부분의존성도표 등은 볼 수 있다.
4. neural network : 인공신경망 분석은 반복적인 학습과정을 거쳐 데이터에 숨어져 잇는 패턴을 찾아내는 모델링 기법이다. 자료분석 분야에서 인공신경망은 복잡하고 비선형적인 구조를 가진 자료에서 예측 문제를 해결하기 위해 사용되어 진다. 특히 예측기법으로써 ANN의 연구가 계속 되고 있는 점이 특징이다. ANN은 입력층, 은닉층, 출력층으로 총 3개의 층으로 구성되어 있다. 결합함수, 활성함수 등이 각 층에서 적용되어지며, 은닉층이 1개 이상 있다면, 다층 신경망이라고 할 수 있다. 모수들이 굉장히 많은 것이 특징인데, 주로 역전파 알고리즘을 사용하여 모수들을 추정한다.
5. CART : Breiman에 의해 제안된 Classification and regression tree 알고리즘은 불순도를 분리기준으로 사용하여 연속형 또는 범주형의 목표변수들을 분석할 수 있는 이지분리방법이다. 전체탐색법을 이용하여 분류변수와 분류집합의 선택을 동시에 결정하였다. 분류는 부모노드보다 각각의 자식노드간에 불순도에 차이가 뚜렷이 나타나는 방법을 제안하였다. 즉 범주형 목표변수인 경우 지니 지수, 엔트로피 지수가 사용되며, 연속형 목표변수인 경우에는 분산의 감소량을 사용한다. CART의 단점은 목표변수의 범주의 수가 많은 경우에는 시간이 오래 걸리지만, 이번 프로젝트는 이진분류이므로 사용 하기에 부담이 없었다.
6. CHAID : chi-squared automatic interaction detection 는 kass에 의해 제안되었고, 범주형 목표변수이면 카이제곱 검정통계량을, 연속형 목표변수이면 F통계량을 이용하여 분리 또는 병합을 반복하여 다지분리를 수행하는 알고리즘이다. CHAID는 목표변수가 범주형 자료일 때, Pearson의 카이제곱 통계량 또는 우도비 카이제곱 통계량을 이용하여 분리를 수행한다. 만약, 목표변수가 순서형 자료 또는 사전 그룹화된 연속형 자료인 경우에는 우도비 카이제곱 통계량을 사용한다. 단점으로는 설명변수의 남아있는 범주가 통계적으로 유의하게 되면 더 이상 병합을 하지 않는다. 그 결과 최적의 분리를 하지 못하는 경우가 생기는데 Exhaustive CHAID는 모든 조합을 탐색하여 최적분리를 찾아내기 때문에, 단점을 보완하였지만, 계산량이 많아지기 때문에 프로젝트에서는 CHAID를 채택하였다.
7. CTREE : conditional inference tree는 hothorn에 의해 제안되었다. 특징은 일변량 회귀모형뿐만 아니라 다변량 회귀모형에도 수행할 수 있다. CTREE알고리즘은 매우 특이하게(전의 알고리즘들은 전체탐색법을 시행) 분류변수선택과 분리 절차를 따로 진행한다. 목표변수와 설명변수들간에 선형 정도를 구성하고, 순열검정을 이용하여 목표변수와 설명변수 간에 연관성을 살펴보고 특정한 유의수준 하에서 p-value들을 계산한다. 마지막으로 목표변수와 가장 강한 연관성이 있는 설명변수를 찾아, 그 변수를 분류변수로 채택한다.

#### * 의사결정나무
데이터 마이닝에 널리 쓰이는 방법으로서 나무구조로 도표화하여 예측과 분류를 수행하는 분석방법이다. Decision Tree는 표본 집단을 몇 개의 소그룹으로 분리하고, 새로운 관측치의 범주를 예측하는 분류기법이라고 할 수 있다. 목표변수의 분포를 구별하는 정도를 순수도, 불순도 등의 분리기준에 의해서 측정한다. 목표변수가 범주형인 경우 지니 지수, 엔트로피 지수, 카이제곱 통계량 등을 사용하고, 목표변수가 연속형인 경우 분산의 감소량, F-통계량 등을 사용한다.

### 2. 변수의 중요도 파악
1. 너무 많은 컬럼으로 인해 각 모델의 결과로 중요변수들을 선택한다.
2. 파악된 중요변수들로 다시한번 모델을 선정한다.
3. 결과를 해석한다.

### 3. 조합된 모델
#### 1. CHAID
1. logistic regression
2. C4.5
3. randomforest
4. neural network
5. logistic + randomforest

#### 2. CART
1. logistic regression
2. C4.5
3. randomforest
4. neural network
5. logistic + randomforest

#### 3. CTREE
1. logistic regression
2. C4.5
3. randomforest
4. neural network
5. logistic + randomforest

## 4 모델링 평가
### 1. ROC curve(AUC)를 이용한 모델 평가
1. logistic regression + CHAID
2. C4.5 + CHAID
3. no selection + CART
4. logistic regression + randomforest + CART
5. no selection + CTREE
6. C4.5 + CTREE
7. randomforest + CTREE

### 2. Train Data and Validation Data

## 5 결론

