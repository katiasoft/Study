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

위 그림은 RPN의 개념을 시각적으로 보여주지만 실제 작동 방식은 보기보다 이해하기 어렵다.<br> 
좀 더 풀어서 그림으로 표현하면 아래와 같다.

><img src="https://user-images.githubusercontent.com/69491771/93965303-c4668500-fd9c-11ea-9e16-86e0f292e97b.png" width="801" height="386">

RPN이 동작하는 알고리즘은 다음과 같다.

1. CNN을 통해 뽑아낸 Feature map을 input으로 받는다. 이때 Feature map의 크기를 H x W x C로 잡는다. 각각 가로, 세로, 채널이다.

2. Feature map에 3x3 Conv을 256 혹은 512 채널만큼 수행한다. 위 그림에서 intermediate layer에 해당한다. 이때, padding을 1로 설정해주어 H x W가 보존될 수 있도록 한다. intermediate layer 수행 결과 H x W x 256 or H x W x 512 크기의 두 번째 Feature map을 얻는다.

3. 두 번째 Feature map을 입력 받아서 classification과 bounding box regression 예측 값을 계산해주어야 한다. 이 때 주의해야할 점은 Fully Connected Layer가 아니라 1 x 1 컨볼루션을 이용하여 계산하는 Fully Convolution Network의 특징을 갖는다. 

4. 먼저 Classification을 수행하기 위해서 1 x 1 Conv을 (2(오브젝트 인지 아닌지 나타내는 지표 수) x 9(anchor 개수)) 체널 수 만큼 수행해주며, 그 결과로 H x W x 18 크기의 Feature map을 얻는다. H x W 상의 하나의 인덱스는 Feature map 상의 좌표를 의미하고, 그 아래 18개의 체널은 각각 해당 좌표를 anchor로 삼아 k개의 anchor box들이 object인지 아닌지에 대한 예측 값을 담고 있다. 즉, 한번의 1x1 컨볼루션으로 H x W 개의 anchor 좌표들에 대한 예측을 모두 수행한 것입니다. 이제 이 값들을 적절히 reshape 해준 다음 Softmax를 적용하여 해당 anchor가 object일 확률 값을 얻는다.

5. 두 번째로 Bounding Box Regression 예측 값을 얻기 위한 1 x 1 Conv을 (4 x 9) 체널 수 만큼 수행한다. Regression이기 때문에 결과로 얻은 값을 그대로 사용한다.

6. 이제 앞서 얻은 값들로 RoI를 계산해야 한다. 먼저 Classification을 통해서 얻은 물체일 확률 값들을 정렬한 다음, 높은 순으로 K개의 anchor만 추려냅니다. 그 다음 K개의 anchor들에 각각 Bounding box regression을 적용해준다. 그 다음 Non-Maximum-Suppression을 적용하여 RoI을 구해준다.

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
