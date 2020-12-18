# AlexNet

AlexNet은 2012년에 개체된 ILSVRC(ImageNet Large Scale Visual Recognition Challenge) 대회의 우승을 차지한 컨볼루션 신경망(CNN) 구조이다. CNN의 부흥에 아주 큰 역할을 한 구조라고 말할 수 있다.

AlexNet의 Original 논문명 "ImageNet Classification with Deep Convolution Neural Networks"이다, 이 논문의 첫번째 저자가 Alex khrizevsky이기 때문에 그의 이름을 따서 AlexNet이라고 부른다. CNN의 가장 간단한 구조중의 하나인 LeNet-5와 비교하면거 설명하니, LeNet-5을 먼저 읽어 보는것을 추천한다.

## AlexNet 특징

#### (1) ReLU 함수

- 활성화 함수로는 LeNet-5에서 사용 되었던 Tanh함수 대신에 ReLU함수가 사용된다.
- ReLU는 rectified linear unit의 약자입니다.
- 같은 정확도를 유지하면서 Tanh을 사용하는 것보다 6배나 빠르다고 한다.
- AlexNet 이후에는 활성화함수로 ReLU함수를 사용하는 것이 선호되고 있다.

#### (2) DropOut

- 과적합(over-fitting)을 막기위해서 규제 기술의 일종인 DropOut을 사용한다.
- DropOut이란 fully connected layer의 뉴런중 일부를 생략하면서 학습을 진행하는 것이다. 몇몇 뉴런의 값을 0으로 바꿔버린다.
- 따라서 그 뉴런들은 foward pass와 back propagation에 아무런 영향을 미치지 않는다.
- DropOut은 훈련시에 적용되는 것이고, 테스트시에는 모든 뉴런을 사용한다.

#### (3) Overlapping pooling

- CNN에서 polling의 역할은 컨볼루션을 통해 얻은 특성맵을 줄이기 위함이다.
- LeNet-5의 경우 평균풀링(average pooling)이 사용된 반면, AlexNet에서는 최대풀링(max pooling)이 사용되었다.
- 또한 LeNet-5의 경우 풀링 커널이 움직이는 보폭인 stride를 커널 사이즈보다 작게 하는 overlapping pooling을 적용했다.
- 따라서 정확히 말하면 LeNet-5는 non-overlapping 평균 풀링을 사용한 것이고 AlexNet은 overlapping 최대풀링을 사용한 것이다.
- overlapping 풀링을 하면 풀링 커널이 중첩되면서  지나가는 반면, non overlapping 풀링을 하면 중첩없이 진행된다.
- overlapping 풀링이 top-1, top-5 에러율을 줄이는데 좀 더 효과가 있다고 한다.

#### (4) local response normalization

- 신경 생물학에는 lateral inhibition이라고 불리는 개념이 있다.
- 활성화된 뉴런이 주변 이웃 뉴런들을 억누르는 현상을 의미한다. 
- lateral inhibition 현상을 모델링한 것이 바로 local response normalization 이다.
- 강하게 활성화된 뉴런의 주변 이웃들에 대해서 noramalization을 실행한다.
- 주변에 비해 어떤 뉴런이 비교적 강하게 활성화 되어 있다면, 그 뉴런의 반응은 더욱 더 돋보이게 될것이다.
- 반면 강하게 활성화된 뉴런 주변도 모두 강하게 활성화 되어 있다면, local response normalization 이후에는 모두 값이 작아질 것이다.

#### (5) Data augmentation

- 과적합을 막기위해 DropOut 말고도 또다른 방법이 사용되었다.
- 과적합을 막는 가장 좋은 방법중 하나는 데이터의 양을 늘리는 것이다.
- 훈련시킬때 적은양의 데이터를 가지고 훈련시킬 경우 과적합 가능성이 크기 때문이다.
- 따라서 AlexNet의 개발자들은 Data augmentation이란 방법을 통해 데이터의 양을 늘렸다.

## AlexNet 구조

AlexNet의 기본구조는 LeNet-5와 크게 다르지 않다, 2개의 GPU로 병렬연산을 수행하기위해서 병렬적인 구조로 설계되었다는 점이 가장 큰 변화이다. 

<img width="622" alt="스크린샷 2020-12-16 오후 1 50 28" src="https://user-images.githubusercontent.com/71860142/102306604-f5261700-3fa5-11eb-9cf4-08b9930b31bf.png"> 

AlexNet은 8개의 레이어로 구성되어 있다, 5개의 컨볼루션 레이어와 3개의 full-connected 레이어로 구성되어 있다. 두번째와 네번째, 다섯번째 컨볼루션 레이어들은 전단계의 같은 채널의 특성맵들과만 연결되어 있는 반면, 세번째 컨볼루션 레이어는 전 단계의 두 채널의 특성맵들과 모두 연결되어 있다는 것을 집고 넘어가자.

이제 각 레이어마다 어떤 작업이 수행되는지 살펴보자, 우선 AlexNet에 입력 되는 것은 227 x 227 x 3 이미지다.(227 x 227 사이즈의 RGB 컬러이미지를 뜻한다.) 그림에는 224로 되어 있는데 잘못된 것이다.

#### (1) 첫번째 레이어(컨볼루션 레이어) 

- 96개의 11x11x3 사이즈 필터 커널로 입력 영상을 컨볼루션 해준다. 컨볼루션 보폭(stride)를 4로 설정했고, zero-padding은 사용하지 않았다. 결과적으로 55x55x96 특성맵(96장의 55x55 사이즈 특성 맵들)이 산출된다. 
- 그 다음이 ReLU함수로 활성화 해준다. 
- 이어서 3x3 overlapping max pooling이 stride값이 2로 시행된다. 그 결과 27x27x96 특성맵을 갖게된다. 
- 그 다음에는 수렴속도를 높이기위해 Local response normalization이 시행된다. Local response normalization은 특성맵의  차원을 변화 시키지 않으므로, 특성맵의 크기는 27x27x96으로 유지된다.

#### (2) 두번째 레이어(컨볼루션 레이어)

- 256개의 5x5x48 커널을 사용하여 전 단계의 특징맵을 컨볼루션 해준다.
- stride는 1로, zero-padding은 2로 설정했다. 
- 따라서 27x27x256 특성맵(256장의 27x27 사이즈 특성맵들)을 앋게 된다.
- ReLU함수로 활성화 해준다.
- 그 다음에 3x3 overlapping max pooling을 stride를 2로 시행된다. 그 결과 13x13x256 특성맵을 얻게된다.
- 그 후에 local response normalization이 시행되고, 특성맵의 크기는 13x13x256으로  유지된다.

#### (3) 세번째 레이어(컨볼루션 레이어)

- 384개의 3x3x256 커널을 사용하여 전 단계의 특성맵을 컨볼루션해준다.
- stride와 zero-padding 모두 1로 설정한다. 
- 따라서 13x13x384 특성맵(384장의 13x13 사이즈 특성맵들)을 얻게 된다.
- ReLU함수로 활성화 해준다.

#### (4) 네번째 레이어(컨볼루션 레이어)

- 384개의 3x3x192 커널을 사용하여 전 단계의 특성맵을 컨볼루션해준다.
- stride와 zero-padding 모두 1로 설정한다. 
- 따라서 13x13x384 특성맵(384장의 13x13 사이즈 특성맵들)을 얻게 된다.
- ReLU함수로 활성화 해준다.

#### (5) 다섯번째 레이어(컨볼루션 레이어)

- 256개의 3x3x192 커널을 사용해서 전 단계의 특성맵을 컨볼루션 해준다.
- stride와 zero-padding 모두 1로 설정한다. 
- 따라서 13x13x256 특성맵(256장의 13x13 사이즈 특성맵들)을 얻게 된다.
- ReLU함수로 활성화 해준다.
- 그 다음에 3x3 overlapping max pooling을 stride를 2로 시행한다. 그 결과 6x6x256 특성맵을 얻게 된다.

#### (6) 여섯번째 레이어(Fully connected layer)

- 6x6x256 특성맵을 Flatten 해줘서 6x6x256 = 9216 차원의 벡터로 만들어준다.
- 그것을 여섯번째 레이어의 4096개의 뉴런과 Fully connected 해준다. 그 결과를 ReLU함수로 활성화 한다.

#### (7) 일곱번째 레이어(Fully connected layer)

- 4096개의 뉴런으로 구성 되어 있다.
- 전 단계의 4096개 뉴런과 Fully connected 되어 있다.
- 출력값을 ReLU함수로 활성화 한다.

#### (8) 여덟번째 레이어(Fully connected layer)

- 1000개의 뉴런으로 구성 되어 있다.
- 전 단계의 4096개 뉴런과 Fully connected 되어있다.
- 1000개의 뉴런의 출력값에 softmax함수를 적용해 1000개 클래스 각각에 속할 확률을 나타낸다.