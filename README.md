# Faster-R_CNN   
### Process   

<p align="center"><img src="https://github.com/suhyeong-jeon/Faster-R_CNN/assets/70623959/5a440f00-f968-40dc-b917-f10c96932df0"></p>   

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



<hr>

### References   
##### [herbwood - Faster R-CNN Review](https://herbwood.tistory.com/10)
##### [Faster R-CNN Thesis](https://arxiv.org/pdf/1506.01497.pdf)
