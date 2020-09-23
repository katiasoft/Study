# Fast R-CNN
## Introduction
Fast R-CNN은 이전의 R-CNN의 단점을 극복하고자 나왔으며, R-CNN 단점은 다음과 같다.

1. RoI(Region of Interest) 마다 CNN연산을 함으로써 속도가 저하된다.
2. Multi-Stage pipelines으로써 모델을 한번에 학습시키지 못한다.

그리고 Fast R-CNN에서는 다음 두가지를 통해 한계를 극복한다.
1. RoI Pooling
2. CNN 특징 추출부터 Classification, Bounding Box Regression까지의 하나의 모델에서 학습

## Fast R-CNN
><img src="https://user-images.githubusercontent.com/69491771/93858183-b65e2900-fcf6-11ea-83a2-852f5dbcd113.png" width="500" height="240">

Fast R-CNN의 가장 핵심적인 아이디어는 RoI Pooling이다.<br>
R-CNN에서 CNN outputdl FC layer의 input으로 들어가야했기 때문에 CNN input을 동일 size로 맞춰줘야 했다.<br>
따라서 원래 이미지에서 추출한 RoI를 crop, warp을 통해 동일한 size로 조정했었다.<br>
그러나 실제로 'FC layer의 input이 고정인거지 CNN input은 고정이 아니다'<br>
따라서 CNN에는 입력 이미지 크기, 비율 관계없이 input으로 들어갈 수 있다.<br>
즉, FC layer의 input으로 들어갈때만 size를 맞춰주기만 하면 된다. 

### ☞ Fast R-CNN 프로세스
1. R-CNN에서와 마찬가지로 Selective Search를 통해 RoI를 찾는다. 
2. 전체 이미지를 CNN에 통과시켜 feature map을 추출한다.
3. Selective Search로 찾았었던 RoI를 feature map크기에 맞춰서 projection시킨다.
4. projection시킨 RoI에 대해 RoI Pooling을 진행하여 고정된 크기의 feature vector를 얻는다.
5. feature vector는 FC layer를 통과한 뒤에 구 브랜치로 나뉘게 된다.
6. 하나는 softmax를 통과하여 RoI에 대해 object classification을 한다. 
7. bounding box regression을 통해 selective search로 찾은 box의 위치를 조정한다. 

### ☞ Spatial Pyramid Pooling(SPP)

><img src="https://user-images.githubusercontent.com/69491771/93860039-65036900-fcf9-11ea-9511-82959c076067.png" width="380" height="300">

1. 이미지를 CNN에 통과시켜 Feature map을 추출한다.
2. 미리 정해진 4x4, 2x2, 1x1 영역의 피라미드로 Feature map을 나눠준다. 피라미드 한칸을 bin이라 한다.
3. bin내에서 max pooling을 적용하여 각 bin마다 하나의 값을 추출한다.
4. 최종적으로 피라미드 크기만큼 max값을 추출하여 3개의 피라미드의 결과를 이여붙여 고정된 크기 vector를 만든다.

정리하자면, 4x4, 2x2, 1x1 세 가지 피라미드가 존재하고, max pooling을 적용하여 각 피라미드 크기에 맞게 max값을 뽑아낸다.<br>
각 피라미드 별로 뽑아낸 max값들을 이여붙여 고정된 크기 vector를 만들고 이게 FC layer의 input으로 들어간다.

<br>
따라서 CNN을 통과한 Feature map에서 2000개의 Region proposal을 만들고, Region proposal마다<br>
SPPNet에 집어넣어 고정된 크기의 Feature vector를 얻어낸다.

**이 작업을 통해 모든 2000개의 Region proposal마다 해야했던 2000번의 CNN연산이 1번으로 줄어든다.**

### ☞ Fast R-CNN에서 이 SPP가 적용된 구조

><img src="https://user-images.githubusercontent.com/69491771/93861482-8ebd8f80-fcfb-11ea-8033-8fdfa04e20b3.png" width="766" height="438">

실제로 Fast R-CNN에서는 1개의 피라미드를 적용시킨 SPP로 구성되어있다. 또한 피라미드의 사이즈는 7x7이다.<br>
Fast R-CNN에서 적용된 1개의 피라미드 SPP로 고정된 크기의 Feature Vector를 만드는 과정을 **"RoI Pooling"**이라 한다.

### ☞ RoI Pooling

><img src="https://user-images.githubusercontent.com/69491771/93862078-7ef27b00-fcfc-11ea-8233-25c04600e4fd.png" width="620" height="245">

1. Fast R-CNN에서 먼저 입력 이미지를 CNN에 통과시켜 feature map을 추출한다.<br>
2. 그 후 이전에 미리 Selective search로 만들어놨던 RoI(=region proposal)을 feature map에 projection시킨다.<br>
3. 위 그림의 가장 좌측 그림이 feature map이고 그 안에 h x w크기의 검은색 box가 투영된 RoI이다.
    * 미리 설정한 HxW크기로 만들어주기 위해서 (h/H) * (w/H) 크기만큼 grid를 RoI위에 만든다.
    * RoI를 grid크기로 split시킨 뒤 max pooling을 적용시켜 결국 각 grid 칸마다 하나의 값을 추출한다.

4. 위 작업을 통해 feature map에 투영했던 hxw크기의 RoI는 HxW크기의 고정된 feature vector로 변환된다.

<br>
이렇게 RoI pooling을 이용함으로써

<b>"원래 이미지를 CNN에 통과시킨 후 나온 feature map에 이전에 생성한 RoI를 projection시키고</b><br>
<b>이 RoI를 FC layer input 크기에 맞게 고정된 크기로 변형할 수가 있다"</b>

따라서 더이상 2000번의 CNN연산이 필요하지 않고 1번의 CNN연산으로 속도를 대폭 높일 수 있었다.

### ☞ 1-stage : Trainable
* R-CNN의 두번째 문제였던Multi-stage pipeline으로 인해 3가지 모델로 따로 학습해야한다는 점이다.
* R-CNN에서는 CNN을 통과한 후 각각 서로다른 모델인 SVM(Classification), bounding box regression(localization)안으로 들어가 forword 됬기 때문에 연산이 공유 되지 않았다.
* bounding box regression은 CNN을 거치기 전의 regional 데이터가 input으로 들어가고 SVM은 CNN을 거친 후의 Feature map이 input으로 들어가기에 연산이 겹치지 않는다.

><img src="https://user-images.githubusercontent.com/69491771/93858183-b65e2900-fcf6-11ea-83a2-852f5dbcd113.png" width="500" height="240">

* 위 그림을 다시보면 RoI Pooling을 추가함으로써 이제 RoI영역을 CNN을 거친후의 Feature map에 투영시킬 수 있다.
* 따라서 동일 Data가 각자 softmax(Classification), bbox regressor(localization)으로 들어가기에 연산을 공유한다.
* 모델이 1-stage가 되어 한 번에 학습을 시킬 수 있다.

### ☞ Result

RoI Pooling을 하나 추가함으로써
1. CNN후에 Region proposal 연산이 2000번에서 1번의 CNN연산으로 줄어들었다.
2. 변경된 Feature Vector가 결국 기존의 Region proposal을 projection시킨 후 연산한 것이므로 해당 output으로 Classification과 bbox regression도 학습이 가능하다.

그러나 여전히 Fast R-CNN에서도 R-CNN에서와 마찬가지로 RoI를 생성하는 Selective Search 알고리즘은 CNN 외부에서 진행되므로 이부분이 속도를 올리는 데 장애물이 된다.

