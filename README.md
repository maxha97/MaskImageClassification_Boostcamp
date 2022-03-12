# 마스크 착용여부 이미지 분류 대회(Level1)

## 대회 설명
COVID-19의 확산으로 우리나라는 물론 전 세계 사람들은 경제적, 생산적인 활동에 많은 제약을 가지게 되었습니다. 우리나라는 COVID-19 확산 방지를 위해 사회적 거리 두기를 단계적으로 시행하는 등의 많은 노력을 하고 있습니다. 과거 높은 사망률을 가진 사스(SARS)나 에볼라(Ebola)와는 달리 COVID-19의 치사율은 오히려 비교적 낮은 편에 속합니다. 그럼에도 불구하고, 이렇게 오랜 기간 동안 우리를 괴롭히고 있는 근본적인 이유는 바로 COVID-19의 강력한 전염력 때문입니다.

감염자의 입, 호흡기로부터 나오는 비말, 침 등으로 인해 다른 사람에게 쉽게 전파가 될 수 있기 때문에 감염 확산 방지를 위해 무엇보다 중요한 것은 모든 사람이 마스크로 코와 입을 가려서 혹시 모를 감염자로부터의 전파 경로를 원천 차단하는 것입니다. 이를 위해 공공 장소에 있는 사람들은 반드시 마스크를 착용해야 할 필요가 있으며, 무엇 보다도 코와 입을 완전히 가릴 수 있도록 올바르게 착용하는 것이 중요합니다. 하지만 넓은 공공장소에서 모든 사람들의 올바른 마스크 착용 상태를 검사하기 위해서는 추가적인 인적자원이 필요할 것입니다.

따라서, 우리는 카메라로 비춰진 사람 얼굴 이미지 만으로 이 사람이 마스크를 쓰고 있는지, 쓰지 않았는지, 정확히 쓴 것이 맞는지 자동으로 가려낼 수 있는 시스템이 필요합니다. 이 시스템이 공공장소 입구에 갖춰져 있다면 적은 인적자원으로도 충분히 검사가 가능할 것입니다.



## 대회 참여에서 설정한 목표
- 베이스라인 코드 이해를 통해, 어떻게 image classification 태스크가 진행되는지를 알고자 하였다.
- 기존에 사용해보지 못한, Wandb, Tensorboard를 활용하여 성능을 비교하고, 비대면 협업의 효율성을 극대화해고자 하였다.
- 다양한 Loss에 대한 원리를 이해하고, 사용을 통해 학습에 대한 경향성을 파악하여 가장 적합한 학습방안을 제시하고자 하였다.
- Resnet18을 시작으로, 다양한 SOTA 모델을 활용하여 모델 간의 성능을 확인하고 구동의 효율성을 확인하고자 하였습니다.



## 진행하면서 학습하고 시도해본 것들

### 1) 다양한 모델 적용
- **Efficientnet 모델(파라미터 수 및 모델의 크기에 따라 b0~b7버전이 존재)**
![image](https://user-images.githubusercontent.com/53209003/156936117-980d991b-0769-4dcd-8630-e606e8c75b3e.png)   
 상대적으로 동일한 성능을 가진 Resnet 모델들 대비, 훨씬 더 적은 파라미터 수로 동일한 성능을 달성하였습니다. 이를 통해, 20202년 기준 더 적은 파라미터 수를 가짐에도 불구하고 당시의 높은 성능을 자랑하는 AmoebaNet, Inception, DenseNet, Resnet 모델 등을 따돌리며 SOTA 성능을 달성하였습니다.   
 ![image](https://user-images.githubusercontent.com/53209003/156936269-2b789b64-cc7e-4335-a065-cedc44206ee8.png)   
이는 기존의 네트워크 width(모델 채널 수), depth(모델 네트워크 수), resolution(입력 이미지 해상도)를 늘릴 경우 성능이 점차적으로 증가하고 수렴함을 확인할 수 있음. 이 값들이 증가하되, 이 3가지 지표가 무작정 늘리는 것이 좋은 것이 아니라, 서로 간의 관계를 기반으로 최적의 증가값을 찾아주는 것이 비전 모델의 성능에 중요한 영향을 미친다는 것을 확인후 실행에 옮긴 모델입니다.

(Efficientnet 모델을 활용함에 있어 B3~B4 규모급 모델로도 충분히 성능 향상을 볼 수 있었으나, 좀 더 성능을 올리기 위해 모델의 크기를 키우는 것을 고려하였으나 제공하는 서버에서의 램, 메모리 부족으로 인해 실제로 시도를 해볼 수 없었던 점이 아쉬움으로 남습니다. 추후 서비스로 운용함을 고려해볼 때, 단순히 컴퓨팅 파워에 의존하기 보다는 내부 파이프라인의 효율화를 통해서 달성하는 것이 더 올바른 해결방안으로 생각됩니다.)

### 2) 다양한 Loss 학습 및 적용
- **CrossEntropy Loss**   
![image](https://user-images.githubusercontent.com/53209003/156934799-f7ad4c0c-5f38-46af-9342-16a008444f26.png)     
엔트로피 개념이 손실(loss)함수에서 사용된 사례로 볼 수 있음. 만약 이진분류 태스크가 아닌 N개의 라벨 중 하나로 분류하는 태스크일 때, 입력 데이터가 모델을 통과할 경우 소프트맥스 함수에 의해 각 클래스에 속할 확률이 계산됩니다(Q(x)). 실제 값의 경우, 정답인 클래스만 확률이 1이고, 나머지는 모두 0인 확률분포에 해당합니다(P(x)). 이 두 확률분포의 차이를 Cross Entropy 수식 형태로 표현하여 계산할 수 있으며, Cross Entropy를 줄이기 위한 파라미터를 구하는 것이 negative log likelohood를 최소화하는 파라미터를 구하는 것과 동일하다고 볼 수 있습니다.

- **Focal Loss**   
![image](https://user-images.githubusercontent.com/53209003/156935353-b9c58874-17bd-4cf9-b437-147c4f6d568a.png)    
기존의 Cross Entropy의 경우 분류태스크에서 분류할 라벨별로 데이터가 균등한 비율로 존재할 경우 학습에 문제가 없습니다. 하지만, 라벨 비율이 불균형한 데이터에 있어서는 학습에 문제를 일으킬 수 있습니다. 이는, 데이터 라벨 비중이 높아 샘플 비중이 높은 Easy Negative 샘플에 대해 중점적으로 학습하게되기 때문에 발생합니다. Focal Loss는 기존의 Cross Entropy에 가중치를 곱함으로써, Easy Negative 샘플에 대한 학습에 중점적으로 뒀던 학습과정에 패널티를 부여할 수 있습니다. 이에 따라, 데이터 라벨 비중이 낮은 Hard Negative 샘플에 대해서도 추가적으로 학습할 수 있는 구조를 취하여 불균형 데이터에 효과적임을 알 수 있었습니다.

- **F1 Loss**   
F1 score가 정밀도(Precision, Positive로 예측한 것중 실제 Positive 비율)와 재현율(Recall, 실제 Positive인 것 중 Positive로 예측한 비율)을 함께 고려한 점수 metric인만큼, F1 Loss 역시 
클래스 별로 정밀도 및 재현율을 함께 고려했을 것으로 생각되나, 좀 더 학습이 필요한 상황입니다. 

(일반적으로 CrossEntropy Loss의 경우, 초기 에폭부터 높은 validation 성능을 보여주었으나, 추후 validation loss는 꾸준히 증가하여 오히려 성능에 있어서 하락을 가져왔습니다. 이는, Focal Loss 역시, CrossEntropy에 비해 좀 더 나은 성능을 보여줄 뿐, 기존의 경향성과 크게 다르지 않았음을 확인할 수 있었습니다. 반대로, F1 Loss의 경우, 타 Loss 대비 한템포 느린 validation 성능 향상을 가져왔으나, 꾸준한 학습에도 validation loss가 상승하는 모습을 보이지 않고 감소하는 경향성을 보여왔습니다. 이를 통해, 학습에 사용하는 시간이 많이 걸린다는 것이 단점이지만, 모델의 성능을 끌어올리는데 F1-loss가 효율적임을 알 수 있었습니다)

### 3)WanDB 사용을 통한 효율적인 모델 비교 및 하이퍼 파라미터 튜닝 
![image](https://user-images.githubusercontent.com/53209003/156936616-585a1e1d-2010-4ec2-80c7-97c39c9d6f71.png)    
Wandb Teams를 구성하여, 실시간으로 각 팀원들이 실행 중인 모델들의 Train,Validation에서의 성능지표(acc, f1-macro, loss)들을 에폭을 기준으로 비교해볼 수 있었습니다. 이릍 통해, 비대면임에도 불구하고 함께 효율적으로 모델 비교 및 선정이 가능하였습니다. 이외에도, Loss, Optimizer, Learningrate 등에 따른 하이퍼 파라미터 튜닝에 있어서도 효율적으로 최적 파라미터를 찾을 수 있었음. 하지만, 제출성능이 Validation 성능과는 차이가 있어, 100% 완벽한 판단을 내리기에는 어려움이 있어 보조적인 판단 지표로 활용하는 것이 적절해보입니다.


## 추후 보완하면 진행하면 좋은 것들
- **공통된 Github 코드를 기반으로 한 협업**  
협업을 진행하는 과정에 있어, 팀원별로 코드가 달라 성능이 제각각이었으며, 각자의 코드에서 장점을 취합하여 통합코드를 기반으로 실험을 진행하지 못하였습니다. 이에 따라, 새로운 시도들에 대한 성능 평가가 객관적으로 이뤄지지 못하였으며, 최종 코드에 새로운 시도들을 적용할지 여부에 대한 결정을 내리기 어려운 상황이 반복되었습니다. 추후에 진행하는 프로젝트에 있어서는 빠르게 코드 통합을 진행하되, 새로운 시도에 대해선 깃허브 내 피처 브랜치를 따로 생성하여 진행하고, 동일한 성능평가 기준(ex. 리더보드 제출, Validation accuracy)을 적용하여 빠르고 효율적인 의사결정이 이뤄질 수 있도록 노력할 것입니다.

- **지속적인 이미지 데이터 EDA를 통한 성능 개선**
이미지 데이터를 정형데이터처럼 꾸준히 데이터를 탐색하는 자세를 가지지 않고 대회에 접근하였습니다. 이에 따라, 데이터 EDA를 통해서 알 수 있거나, 충분히 성능을 개선할 수 있는 부분을 놓치게 되었습니다. 이미지 데이터 역시 추후에는, 라벨 및 이미지에 주어진 메타 데이터를 기반으로 꾸준히 마무리되기 까지 데이터를 탐구하는 자세를 가질 수 있도록 하겠습니다. 



## 추후 시도해보면 좋은 것들
-	**Nivida APEX – AMP**   
엔비디아에서 제공중이며, 현재는 파이토치 기본 라이브러리로 탑재된 APEX 패키지는 AMP(Automatic Mixed Precision) 기능을 제공하고 있습니다. AMP 기능을 활용할 경우 FP16연산과 FP32 연산을 섞어 학습을 진행할 수 있습니다. 이렇게 진행하게 될 경우, 학습 속도에 있어 향상이 있으며, 16bit를 혼용함에 따라 램사용량을 최적화할 수 있을 것으로 기대됩니다. 이에 따라, 기존에는 네이버 V100 서버에서의 90기가램에서 설정하지 못한 배치사이즈를 시도해보거나, 더 많은 파라미터 수를 가진 모델(ex. Efficientnet_b7)을 시도해 봄으로써 성능 향상을 기대해볼 수 기대됩니다.

-	**Pytorch Lightning**   
파이토치의 하이레벨 언어로 모델학습 및 예측 과정에서 훨씬 가벼운 코드를 작성할 수 있도록 도와줍니다. 추후, 코드 작성에 소모되는 시간을 단축하기 위해 시도해보면 좋을 것으로 생각됩니다. 
([파이토치] 머신러닝 Pytorch 모델의 성능을 극대화하는 7가지 팁!: https://bbdata.tistory.com/9)
