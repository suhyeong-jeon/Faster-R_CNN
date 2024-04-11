# Faster-R_CNN   
### Process   

<p align="center"><img src="https://github.com/suhyeong-jeon/Faster-R_CNN/assets/70623959/5a440f00-f968-40dc-b917-f10c96932df0"></p>   

#####   
##### 1. 원본 이미지를 pre-trained된 CNN 모델에 입력해 feature map을 얻는다.
##### 2. feature map은 RPN(Region Proposal Network)에 전달되 적절한 region proposal을 산출한다.
##### 3. region proposal과 1. 과정에서 얻은 feature map을 통해 RoI pooling을 수행해 고정된 크기의 feature map을 얻는다.
##### 4. Fast R-CNN 모델에 고정된 크기의 feature map을 입력하여 Classification과 Bounding Box Regression을 수행한다.
<hr>

## Main Ideas   
### 1. Anchor Box   
##### 원본 이미지를 일정 간격의 grid로 나눠 각 grid cell을 bounding box로 간주해 feature map에 encode하는 Dense Sampling 방식 사용.   
##### 이때 sub-sampling ratio를 기준으로 grid를 나누게 된다. 예를들어 원본 이미지의 크기가 800 * 800 이고 sub_sampling ratio가 1 / 100 이면, CNN 모델에 입력해 얻은 최종 feature map의 크기는 8 * 8(800 * 1 / 100)가 된다.   
##### 이때 feature map의 각 cell은 원본 이미지의 100 * 100 만큼의 영역에 대한 정보를 함축하고 있다. 800 * 800 크기의 이미지에 대해서는 8 * 8개의 bounding box가 생성된다.   
#####   
##### 하지만 고정된 크기의 bounding box를 사용하면 다양한 크기의 객체를 포착하지 못할 수 있는 문제가 발생하게 된다. Faster-R-CNN 논문에서는 이러한 문제를 해결하고자 지정한 위치에 사전에 정의한 서로 다른 scale(크기)와 aspect ratio(가로세로비)를 가지는 bounding box인 Anchor Box를 생성하여 다양한 크기의 객체를 포착하는 방법을 제시한다. 본 논문에서는 scale([128, 256, 512)]과 aspect ratio([1:1, 1:2, 2:1)]를 가지는 총 9개의 서로 다른 anchor box를 사전에 정의했다.   
#####   
##### Anchor box는 원본 이미지의 각 grid cell의 중심을 기준으로 생성된다. 그리고 sub-sampling ratio를 기준으로 anchor box를 생성하는 기준 anchor를 정한다. 해당 anchor를 기준으로 사전에 정의한 aspect ratio에 의해 9개의 anchor box를 생성한다.   
##### 예를 들어 600 * 800 크기의 이미지가 있고 sub-sampling ratio는 1/16이라고 한다면, 생성되는 anchor의 수는 (600 * (1/16) * 800 * (1/16)) = 1900 개가 된다. 그리고 anchor box는 1900 * 9 = 17100 개가 만들어지게 된다.   
##### 이렇게 고정된 크기의 bounding box를 사용할때 보다 9배 많은 bounding box를 생성하며, 보다 다양한 크기의 객체를 감지하는것이 가능해진다.   
#####   
### 2. RPN(Region Proposal Network)   
##### RPM이란 원본 이미지에서 region proposals을 추출하는 네트워크다. 원본 이미지에서 anchor box를 생성하면 많은 region proposals가 만들어지는데, RPN은 region proposals에 대해 class score를 부여하고, bounding box coefficient를 출력하는 기능을 한다.   
#####   
<p align="center"><img src="https://github.com/suhyeong-jeon/Faster-R_CNN/assets/70623959/4affc751-de68-4b63-9169-eb410f39ab1f"></p>

#####   
##### 1. 원본 이미지를 pretrained된 VGG 모델에 입력하여 feature map을 얻는다.   
##### 원본 이미지의 크기가 800 * 800이고, sub-sampling ratio가 1 / 100이라고 한다면, 8 * 8 * 512 크기의 feature map이 생성됨.   
#####    
##### 2. 얻은 feature map에 크기가 유지되도록 padding을 추가한 후, 3 * 3 conv 연산을 적용. ((F - C)/stride + 1), 즉 ((8+2) - 3 / 1) + 1 = 8   
##### 8 * 8 * 512 feature map에 대하여 3 * 3 연산을 적용해 8 * 8 * 512의 feature map이 출력.   
#####   
##### 3. class score를 부여하기 위해 feature map에 대해 1 * 1 conv 연산을 적용한다. 이때 출력되는 feature map의 channel 수는 2 * 9 가 되도록 설정한다. RPN에서는 후보 영역이 어떤 class에 해당하는지까지 구체적인 분류를 하지 않고 객체가 포함되어 있는지 여부만을 분류함. 또한 anchor box를 각 grid cell마다 9개가 되도록 설정했기 때문에 channel 수는 2(object 여부) * 9(anchor box 9개)가 된다.   
##### 8 * 8 * 512 크기의 feature map을 입력받아 8 * 8 * 2 * 9 크기의 feature map 출력.
#####   
##### 4. bounding box regressor를 얻기 위해 feature map에 대하여 1 * 1 conv 연산을 적용한다. 이때 출력되는 feature map의 channel 수가 4(bounding box regressor) * 9(anchor box 9개)가 되도록 설정한다.   
##### 8 * 8 * 512 크기의 feature map을 입력받아 8 * 8 * 4 * 9 크기의 feature map 출력.   
#####    

<p align="center"><img src="https://github.com/suhyeong-jeon/Faster-R_CNN/assets/70623959/9893391d-3dbd-485f-addd-0c6e4063685d"></p>

#####   
##### 결과적으로 8 * 8 grid cell마다 9개의 anchor box가 생성되어 총 576(8 * 8 * 9)개의 regional proposals가 추출되고, feature map을 통해 각각에 대한 객체 포함 여부와 bounding box regressor를 파악할 수 있다. 이후 class score에 따라 N개의 region proposals만을 추출하고 Non maximum supression을 적용하여 최적의 region proposals만을 Fast R-CNN에 전달하게 된다.   
#####   

### 3. Multi-Task Loss   
##### RPN과 Fast R-CNN을 학습시키기 위해 Multi-task loss를 사용함. 하지만 RPN에서는 객체의 존재 여부만을 분류하고, Fast R-CNN에서는 배경을 포함한 class를 분류함.   
#####   

### 4. Training Faster R-CNN   
##### 



<hr>

### References   
##### [herbwood - Faster R-CNN Review](https://herbwood.tistory.com/10)
##### [Faster R-CNN Thesis](https://arxiv.org/pdf/1506.01497.pdf)
