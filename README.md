# Faster-R_CNN   
### Process   
##### 1. 원본 이미지를 pre-trained된 CNN 모델에 입력해 feature map을 얻는다.
##### 2. feature map은 RPN(Region Proposal Network)에 전달되 적절한 region proposal을 산출한다.
##### 3. region proposal과 1. 과정에서 얻은 feature map을 통해 RoI pooling을 수행해 고정된 크기의 feature map을 얻는다.
##### 4. Fast R-CNN 모델에 고정된 크기의 feature map을 입력하여 Classification과 Bounding Box Regression을 수행한다.
<hr>
