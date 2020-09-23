# Faster R-CNN

## Introduction

### ☞ R-CNN에서는 Multi-stage pipeline으로 진행된다.
1. Region proposal 추출
2. 각 Region proposal별로 CNN연산
3. Classification
4. Bounding Box Regression

### ☞ Fast R-CNN에서는 1-stage pipeline으로 진행된다.
1. Region proposal 추출
2. 전체 Image CNN 연산 
3. RoI Projection, RoI Pooling
4. Classification & Bounding Box Regression

그러나 여전히 Region proposal인 Selective Search알고리즘을 CNN외부에서 연산이므로 RoI 생성단계가 병목이다.<br>
<b>따라서 Faster R-CNN에서는 Region proposal의 Selective Search를 쓰지않고 CNN을 접목하여 속도를 향상시킨다.</b>

## Faster R-CNN

Selective search는 연산 CPU에서 실행되므로 속도가 느리다, 이점을 보완하기위해 Region proposal 생성하는<br> 
네트워크도 GPU에 넣기위해 Conv Layer에서 생성하도록 한 것이 Faster R-CNN이다.


* Faster R-CNN은 한마디로 RPN(Region Proposal Network) + Fast R-CNN이라 할 수 있다. 
* Faster R-CNN은 Fasr R-CNN구조에서 Conv Feature map과 RoI Pooling사이에 RoI를 생성하는 RPN이 추가된 구조이다.

><img src="https://user-images.githubusercontent.com/69491771/93954591-fff75400-fd88-11ea-8634-388d5a48040a.png" width="455" height="415">

* Faster R-CNN에서는 RPN 네트워크에서 사용할 CNN과 Fast R-CNN에서의 Classification, bbox regression을 위해<br>
사용한 CNN 네트워크를 공유하자는 개념에서 나왔다.

><img src="https://user-images.githubusercontent.com/69491771/93955915-ce33bc80-fd8b-11ea-8fe6-d1d5e6659cb6.png" width="616" height="262">

* Faster R-CNN 프로세스이다.
    1. 위 그림에서와 같이 CNN을 통과하여 생성된 Conv Feature map이 RPN에 의해 RoI를 생성한다.
    2. 주의해야할 것이 생성된 RoI는 Feature map에서으 RoI가 아닌 기존 Image에서의 RoI이다.
    3. 기존 Image위에서 생성된 RoI는 Conv Feature map의 크기에 맞게 Rescaling된다.
    4. Feature map에 RoI가 투영되고 나면 FC layer에 의해 Classification과 bbox regression이 수행된다.

**요즘에는 이 FC layer 대신 GAP(Global Average Pooling)을 사용하는 추세이다.**

## RPN (Region Proposal Network)

><img src="https://user-images.githubusercontent.com/69491771/93962165-173c3e80-fd95-11ea-9ff8-cac4859d4980.png" width="426" height="262">

* RPN의 input값은 이전 CNN모델에서 뽑아낸 Feature map이다.
* Region proposal을 생성하기 위해 Feature map위에 nxn widow를 sliding window시킨다.
* 이때, object의 크기와 비율이 어떻게 구성될지 알 수 없기 때문에 k개의 anchor box를 미리 정의해놓는다.
* anchor box가 bounding box가 될 수 있는 것이고 미리 가능할만한 box모양 k개를 정의해놓는 것이다.
* 여기서는 가로 세로 길이 3종류 x 비율 3종류 = 9개의 anchor box를 이용한다.
* 9개의 anchor box를 이용하여 Classification과 bbox regression을 먼저 구한다.


## Non-Maximu Suppression

Faster R-CNN에 대한 학습이 완료된 후 RPN모델을 예측시키며 한 객체당 여러 proposal값이 나올것이다.

이 문제를 해결하기 위해 NMS알고리즘을 사용하여 Proposal의 개수를 줄인다.

여기서 쓰이는 NMS알고리즘은 다음과 같다.
1. box들의 score(Confidence)를 기준으로 정렬한다.
2. score가 가장 높은 box부터 시작해서 다른 모든 box들과 IoU를 계산해서 0.7이상이면 같은 객체를 탐지한 box라고 생각할 수 있기 때문에 해당 box는 지운다.
3. 최종적으로 각 odject별로 Score가 가장 높은 box 하나씩만 남게된다.

><img src="https://user-images.githubusercontent.com/69491771/93964082-b2371780-fd99-11ea-91b3-c891128ce377.png" width="423" height="521">

## Result 

><img src="https://user-images.githubusercontent.com/69491771/93855911-3f736100-fcf3-11ea-87f9-9047ed45aa7f.png" width="575" height="300">

Faster R-DNN은 Fast R-CNN의 단점인 Selective Search 속도 저하를 RPN을 접목시켜서 해결함으로써 위에 그림처럼 R-CNN의 종류중에 제일 속도를 빠르다.

이후의 Mask R-CNN이 나오는데 기존의 모델들처럼 Classification보다는 Image Segmentation에 많이 사용된다.
