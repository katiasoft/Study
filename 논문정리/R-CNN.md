# R-CNN 논문 정리
## INTRO
### 우선 R-CNN을 들어가기 앞서서 Object Detection에 대해 알아보자.

Object Detection이란 이미지가 무엇인지 판단하는 Classification과 이미지 내의 물체의 위치 정보를 찾는 Localization을 수행하는 것을 말한다, 이를 통해 영상 내의 객체가 사람인지 동물인지 물건인지 등을 구별하여 각 객체가 어디에 위치하는지 표시하는 것이 가능하다.

## Introduction
><img src="https://user-images.githubusercontent.com/69491771/93846988-9facd780-fce0-11ea-97bd-79816db89986.png" width="795" height="275">

### R-CNN 프로세스
1. Image를 Input으로 받는다.
2. Selective search알고리즘에 의해 Region proposal output이 약 2000개를 추출한다, 추출한 Region proposal output을 모두 동일 input size로 만들어주기 위해 warp해준다.
3. 2000개의 Warped image를 각각 CNN모델에 넣는다.
4. 각각의 Convolution 결과에 대해 Classification을 진행하여 결과를 얻는다.

## Object detection with R-CNN
### 1. Region proposal(영역 찾기)

><img src="https://user-images.githubusercontent.com/69491771/93848580-f9170580-fce4-11ea-9053-ae603bf445e4.png" width="550" height="355">

R-CNN에서는 가장 먼저 Region proposal단계에서 '물체가 있을 법한 영역'을 찾는다.<br>
기존의 Sliding window방식의 비효율성을 극복하기 위한 것이다. 

#### ☞ Sliding window

><img src="https://user-images.githubusercontent.com/69491771/93849651-8a877700-fce7-11ea-8d01-d6705e9deb3c.png" width="705" height="290">

* Sliding window방식은 이미지에서 물체를 찾기 위해 window의 (크기, 비율)을 임의로 마구 바꿔가면서 모든 영역에 대해서 탐색하는 것이다. 
* 임의의 (크기, 비율)로 모든 영역을 탐색하는 것은 너무 느리다.
* R-CNN에서는 이 비효율성을 극복하기 위해 Selective Search 알고리즘을 사용한다.

#### ☞ Selective Search

><img src="https://user-images.githubusercontent.com/69491771/93849986-27e2ab00-fce8-11ea-877c-5c229de4a684.png" width="705" height="315">

1. 색상, 질감, 영역크기 등을 이용해 non-object-based segmentation을 수행한다. 이 작업을 통해 좌측 제일 하단 그림과 같이 많은 small segmented areas를 얻을 수 있다.
2. Bottom-up방식으로 small segmented areas를 합쳐서 더 큰 segmented areas를 만든다.
3. (2) 작업을 반복하여 최종적으로 2000개의 Region proposal을 생성한다.

### 2. CNN

><img src="https://user-images.githubusercontent.com/69491771/93852165-801bac00-fcec-11ea-814f-7a06121660ca.png" width="550" height="355">


#### ☞ Feature extraction
* 우선 위에서 언급한 Selective Search를 통해 도출 된 각 Region rpoposal로부터 CNN을 사용하여 4096차원의 Feature vector를 추출한다.
* Feature들은 5개의 Convolutional layer와 2개의 Fully connected layer로 전파되는데, 이때 CNN의 입력으로 사용되기 위해 각 Region은 227x227 RGB의 고정 사이즈로 변환되게 된다.
* 각각의 다른 크기를 변환하다보니 위치정보들의 왜곡이 발생한다.

#### ☞ Training
* 학습에 사용되는 CNN모델의 경우 ILSVRC 2012 데이터 셋으로 미리 학습된 pre-trained CNN(AlexNet)모델을 사용한다.

><img src="https://user-images.githubusercontent.com/69491771/93853179-56fc1b00-fcee-11ea-8684-447ee8df0014.png" width="600" height="314">

### 3. SVM & Bounding Box Regression

><img src="https://user-images.githubusercontent.com/69491771/93853424-ceca4580-fcee-11ea-98c8-3b9815532ea2.png" width="550" height="355">

#### ☞ SVM(Support Vector Machine)
* Classification에 최적화된 CNN 모델을 새로운 Detection 작업 그리고 VOC 데이터셋에 적용하기 위해 오직 VOC의 Region proposal를 통해 SGD(stochastic gradient descent)방식으로 CNN 파라미터를 업데이트 한다.
* CNN을 통해 나온 feature map은 SVM을 통해 classification 및 bounding regreesion이 진행되게 되는데, 여기서 SVM 학습을 위해 NMS(non-maximum suppresion)과 IoU(inter-section-over-union)이라는 개념이 활용된다.
* IoU는 Area of Overlap(교집합) / Area of Union(합집합)으로 계산되며, 간단히 말해 전체 bounding box 영역 중 겹치는 부분의 비율을 나타내는데 NMS 알고리즘이 이 IoU 점수를 활용하여 겹치는 박스를 모두 제거하고 가장 적합한 박스만 남기게 된다.
* NMS(Non-maximum suppresion)
    1. 예측한 bounding box들의 예측 점수를 내림차순으로 정렬
    2. 높은 점수의 박스부터 시작하여 나머지 박스들 간의 IoU를 계산
    3. IoU값이 지정한 threhold보다 높은 박스를 제거
    4. 최적의 박스만 남을 떄까지 위 과정을 반복
*  SVM 학습을 위한 라벨로서 IoU를 활용하였고 IoU 가 0.5이상인 것들을 positive 객체로 보고 나머지는 negative로 분류하여 학습하게 된다. 

#### ☞ Bounding Box Regression

><img src="https://user-images.githubusercontent.com/69491771/93854514-dd196100-fcf0-11ea-9d86-88ee30835461.png" width="445" height="301">

* Selective search로 만든 Bounding Box는 정확하지 않기 때문에 물체를 정확히 감싸도록 조정해주는 Bounding Box Regression(선형회귀 모델)을 사용한다.

><img src="https://user-images.githubusercontent.com/69491771/93854522-e1457e80-fcf0-11ea-80de-f345baf7b2e2.png" width="755" height="258">

* 위의 수식에 x,y,w,h는 각각 Bounding Box의 x,y 좌표(위치), width(너비), weight(높이), P는 선택된 Bounding Box이고 G는 Ground truth(실제값) Bounding Box를 의미한다.

## RESULT

><img src="https://user-images.githubusercontent.com/69491771/93855700-e3a8d800-fcf2-11ea-8cf4-5366b958798b.png" width="996" height="243">

* 위 테이블은 VOC 2010 테스트 데이터에 대한 각 모델별 결과이다. 맨 오른쪽에서 mAP를 확인할 수 있는데, 논문에서는 결과를 비교하는데 같은 region proposal 알고리즘을 적용한 UVA모델과 mAP를 비교한다.
* 위 표를 보면 UVA 모델의 mAP는 35.1%이고, R-CNN의 mAP는 53.7%인 것을 확인할 수 있으며 이것은 높은 증가율이라고 저자는 말한다. 또한 VOC 2011/12 데이터 셋 또한 53.3% mAP 높은 성능을 나타냈다.

## SUMMARY
### ☞ R-CNN의 과정은 다음과 같다.
1. R-CNN은 selective search를 통해 region proposal을 먼저 뽑아낸 후 CNN 모델에 들어간다.
2. CNN모델에 들어가 feature vector를 뽑고 각각의 class마다 SVM로 classification을 수행한다.
3. localization error를 줄이기 위해 CNN feature를 이용하여 bounding box regression model을 수정한다.

### ☞ R-CNN의 단점

><img src="https://user-images.githubusercontent.com/69491771/93855911-3f736100-fcf3-11ea-87f9-9047ed45aa7f.png" width="575" height="300">

* 여기서 selective search로 2000개의 region proposal을 뽑고 각 영역마다 CNN을 수행하기 때문에 CNN연산 * 2000 만큼의 시간이 걸려 수행시간이 매우 느리다. 
* Multi-Stage Training을 수행하며, 
* CNN, SVM, Bounding Box Regression 총 세가지의 모델이 multi-stage pipelines으로 복잡한 구조를 가진다. 
* 각 region proposal 에 대해 ConvNet forward pass를 실행할때 연산을 공유하지 않기에 end-to-end 로 학습할 수 없다.
* 따라서 SVM, bounding box regression에서 학습한 결과가 CNN을 업데이트 시키지 못한다.
