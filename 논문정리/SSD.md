# SSD : Single Shot MultiBox Detector

## Abstract.

단일의 Deep Neural Network로 object detection을 구현했다. SSD라고 불리는 접근법은 다양한 크기와 비율을 가진 default box들로 각 feature map에서 bounding box를 뽑아낸다. predict를 할 때는 네트워크가 각 default box가 각각의 사물 카테고리에 속하는 score와 사물모양에 잘 맞는 box를 만들어 낸다. 게다가, 다양한 크기를 가지는 사물을 매끄럽게 변환한 다양한 feature map들을 결합하여 predict에 사용한다. 쉽게보면 object proposal들을 사용하는 방법들과 다르다. 그이유는 proposal을 생성하는 부분과 pixel 또는 feature를 리샘플링하는 부분이 제거했으며 모든 계산을 단일 네트워크에서 진행한다. 단일 네트워크로 학습을 쉽게 할 수 있고 detection에 필요한 요소를 통합했다.

결과로는 VOC2007에서 74.3%의 정확도(mAP)와 59fps가 나왔다고 한다. 비교하자면 최신 Faster R-CNN모델보다 뛰어난 성능이다.

## 1. Introduction
현재 최신 object detection 시스템에서 bounding box가설하기, pixel 또는 feature resample하기, 높은 퀄리티의 classifier 적용시키는 다양한 접근법이 있다.

당시에 SOTA는 Faster R-CNN이었다. 정확도도 높고 deep한 Network이다. 하지만 계산량이 너무 많으며 좋은 하드웨어를 사용해도 실시간으로 사용하기 어렵다. 제일 빨라봤자 7fps밖에 나오지 않았다. 따라서 속도를 올려고 노력했지만 속도가 올라간 만큼 정확도가 감소하였다.

본 논문은 object proposal를 사용하지 않고 object detection을 할 수 있으며 VOC2007 test를 기준으로 Faster R-CNN의 경우 7fps, 73.2%이며, YOLO의 경우가 45fps, 63.4%이다. 본 논문에서 소개하는 SSD의 경우 59fps, 74.3%의 결과를 가져왔다고 한다. 즉, 속도와 정확도를 동시에 잡았다고 할 수 있다.

우선 proposal생성과 reampling단계를 제거하여 속도를 증가시켰다. 또한 정확도를 위해 다른 scale과 aspect ratio를 가지는 default box를 사용하였다. (Faster R-CNN의 Anchor와 유사) 그 후에 다른 크기들의 feature map을 prediction에 사용하였다.

+ YOLO 보다 빠르고 Faster R-CNN보다 정확한 SSD를 소개한다.
+ SSD 핵심은 작은 conv filter들을 사용한 default box들을 여러 feature map에 적용시켜 score와 box 좌표를 예측한다.
+ detection 정확도를 높이기 위해서 여러 크기의 다른 feature map들로부터 여러 크기의 predict를 수행하고 비율 또한 다르게 적용했다.
+ end-to-end 학습을 할 수 있게 구축했으며 저해상도 이미지에서도 높은 정확도를 가진다.
+ 여러대회(PASCAL VOC, COCO, ILSVRC)에서 실험을 진행한 것을 소개한다.

## 2. The Singe Shot Detector(SSD)
<img src="https://user-images.githubusercontent.com/69491771/95286228-ba18b080-089d-11eb-8053-ee2255f6123e.png" width="726" height="272">

위 그림은 SSD Framework로 (a) 훈련하는 동안 가 object에 대한 input image와 ground truth box만 필요로 한다.

convolutional방식에서 다른 크기의 (4x4 그리고 8x8) feature map의 각 위치에 다른 비율의 default box set으로 평가한다. 각 default box에서 shape offset과 모든 object 카테고리에 대한 confidence를 예측한다.

### 2.1 Model 
SSD는 NMS(non maximum suppression)를 거쳐 최종적으로 나오는 bbox와 score를 포함한 box들을 생각하는 feed-foward convolutional natwork(FFCNN)를 기반한다. 밑의 그림은 단순한 FFCNN이다.

> ### Multi-scale feature maps for detection

base network에서 끝이 생략된 conv feature layer를 추가했다. 이 layer들을 점진적으로 크기가 줄어들고 다양한 크기에서 prediction 하도록 했다. detection model은 Overfeat 그리고 YOLO 같이 단일 크기의 feature map을 만드는 다른 feature layer이다.

> ### Convolutional predictors for detection

각각 더해진 feature layer은 prediction을 위해 conv filter가 사용된 세트를 만든다. 즉, 밑의 사진에서 SSD 네트워크 구축의 윗단을 가르킨다.

m x n x p의 feature layer을 얻기 위해 각 convlayer로 부터 3 x 3 x p의 conv layer을 추가로 통과시킨다. 즉, m x n의 위치에 3 x 3 x p 의 kernel을 적용시킨다. 이것을 detedction에 사용한다. bounding box는 default box의 위치로 알 수 있다.

<img src="https://user-images.githubusercontent.com/69491771/95289670-4e871100-08a6-11eb-961e-e32cabdaf330.png" width="797" height="430">

위의 사진은 SSD와 YOLO모델의 비교로 SSD모델은 base network의 끝에 다른 크기와 비율의 box offset을 예측하는 몇개의 feature layer를 추가한다.

### 2.2 Training

> ### Maching strategy

학습을 하는 동안, default box는 gt box와 대응하여 network가 학습된다. 따라서, default box들의 위치, 크기 그리고 비율은 다양하게 선택되어야 한다. MultiBox의 최고 jaccard overlap과 함께 gt box와 default box를 매칭 시켰다. MultiBox와는 다르게 threshold인 0.5보다 크게 잡았다. 이렇게 하면 좀 더 정확하게 겹쳐야 default box가 선택이 된다. 즉, 학습을 좀 더 단순화시킨다.

> ### Choosing scales and aspect ratios for default boxs

다른 크기와 비율의 object를 인식하는 방법에는 여러 가지가 있다.
본 논문에서는 여러 가지의 크디와 비율을 가진 default box들을 이용한다. 다른 ㅏ원의 feature map들을 사용한 네트워크는 성능이 좋다. SSD 또한 다른차원의  feature map을 사용한다. 위의 Convolutional predictors for detection의 그림에서와 같이 다른 차원의 feature map에서 default box를 가져와 인식에 사용한다.

> ### Hard negative mining

matching 단계가 지난 후에는 대부분의 default box들은 negative 일 것입니다. 이것은 positive와 negative 학습데이터의 언벨런스 문제오 이어진다. 따라서 모든 negative 데이터를 사용하는 것이 아니라 negative : positive = 3 : 1로 사용한다. 본 논문을 제작한 분들이 제일 optimal 하다고 한다.

>### Data Augmentation

input으로 사용할 object는 다양한 크기와 모양이 필요하다. 따라서 학습이미지는 다음 3가지중에 하나를 sample한다.
- 전체적인 original input 이미지를 사용한다.
- 0.1, 0.3, 0.5, 0.7 또는 0.9의 Jaccard overlap을 사용한다.
- 무작위로 sample들을 사용한다.

## Experimantal Results

base network로는 VGG16을 적용시켰다고 한다. 하지만 완벽하게 그대로 사용한 것이 아니라 layer를 조금 변형시켜 적용했다고 한다.

### 3.1 PASCAL VOC2007

<img src="https://user-images.githubusercontent.com/69491771/95694245-e94b6b00-0c6b-11eb-9c2f-a507baec1f7d.png" width="756" height="182">

PASCAL VOC2007 데이터를 사용한 결과로 Fast R-CNN보다 SSD가 성능이 더 좋으며 더 큰 input 이미지는 더 나은 결과를 가져온다.( 작은 object는 적은 정보를 가지고 있기 때문에 큰 object가 좋은데 input 이미지 사이즈를 증가시키는 것은 작은 object를 감지를 향상 시켜준다.)

### 3.2 Model Analysis

<img src="https://user-images.githubusercontent.com/69491771/95694408-da18ed00-0c6c-11eb-9661-3d35dc8ecbec.png" width="723" height="173">

- Data Augmentation이 중요하다. -> Data Augmentation을 적용시켰을 때 성능이 향상 되었다.
- 많은 Default box 모양이 더 좋다. -> box를 예측하는 일이 더 쉬워진다.
- Atrous는 더 빠르다. -> aubsample된 Atrous는 속도를 더 빠르게 한다.
- 여러 feature map을 사용하는 것이 성능을 행상시킨다. -> 아래표에서 6개의 feature map을 사용하는 것이 아닌 마지막 feature map을 사용하지 않고 5개의 feature map을 사용하는 것이 더 좋은 성능이 나온다.

<img src="https://user-images.githubusercontent.com/69491771/95694606-f36e6900-0c6d-11eb-98c2-ecd65db6a689.png" width="723" height="262">

## 4. Conclusion 

SSD는 multiple categorise 1-stage detector이다. 제일 큰 특징은 여러 개의 feature map으로부터 다양한 크기의 bounding box를 가져오는 것이다. 이것이 매우 효과적으로 성능 향상에 도움이 되었다. 다른 object detector(faster R_CNN, YOLO등)과 비교하여 높은 인식률과 속도를 보여준다. 마지막으로 R-CNN이나 영상에서 object tracking에서도 잘 사용될 것이다.  
