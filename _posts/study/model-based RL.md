---
title : "Model-based RL"

date : 2023-03-29

categories : rl
---

## DDPG

DDPG는 deep function approximator로 high dimension state spaces와 continuous action spaces에서
policy를 직접 찾는 model free, off-policy, actor-critic 알고리즘이다.


## DPG와 SPG

#### SPG(Stochastic Policy Gradient)

<img src="/surabanke/assets/images/SPG.jpg" width = "400">

각각 state와 action의 분포를 표현하는 함수를 policy, (π) 라고한다.
각 state마다 reward를 최대화 하는 action이 무엇인지 찾고 reward를 최대화하게끔 분포를 찾아야 한다.
π(a|s) = prob(a|s)
그림처럼 꼭 정규분포 모양을 띄지않는다. reward를 최대화하는 mean과 variance에 따라 모양이 달라진다.

#### DPG(Determinisitc Policy Gradient)

<img src="/surabanke/assets/images/delta_func.png" width = "400">

π 함수의 형태가 delta 함수이다. action이 a_t 일때만 reward를 받는다.
델타 함수로 표시해보면
𝛿(a - a_t )로 a_t 만큼 shift시킨 형태이다.
SPG에서 policy의 variance가 0에 수렴하면 DPG와 같아지므로 DPG는 high dimensional action space에 대해 강하다.(?)

이 두개의 policy를 학습시키는 네트워크를 그림으로 그려보면 DPG의 특징을 알 수 있다.

<img src="/surabanke/assets/images/spg_dpg.PNG" width = "400">


## DPG의 policy함수

먼저 total discounted reward, G0 의 expectation형태에서 식이 시작된다.
G0의 expectation은 state s0로 할 수 있는 모든 state를 바꿔가면서 total expected return을 구한다.

<img src="/surabanke/assets/images/ddpg_objectivefunction1.PNG" width = "300">

여기서 v(s0)는 state가 s0로 고정되어있을때의 total return이다.
<img src="/surabanke/assets/images/ddpg_objectivefunction2.PNG" width = "400">

<img src="/surabanke/assets/images/ddpg_objectivefunction3.PNG" width = "450">

이 식은 v(s0)에서 action a0를 따로 빼낸것이다. 노란색으로 표시된 부분을 따로 보면 Q(s0,a0)가 된다.

<img src="/surabanke/assets/images/ddpg_objectivefunction5.PNG" width = "450">

여기서 policy인 P(a0|s0)는 deterministic policy이므로 그냥 action aθ0부분만 곱해지기 때문에 Q(s0,aθ0)가 된다.
action은 deep network를 거쳐 나온 결과이기 때문에 parameterized action이고 올바른 action을 만드는 weight를 업데이트 시키기 위한 gradient방식을 deep deterministic policy gradient라고 한다.

<img src="/surabanke/assets/images/ddpg_objectivefunction5.PNG" width = "450">

DPG는 state에 대한 action을 하고 그 다음 state로 넘어가서 리워드를 주는 것이아니라 action을 한 후 즉시 보상한다.
(R(s0,a0)는 즉시 보상. 그 뒤에 더해지는 것은 state 1부터 끝까지의 reward)

## Deep Determinisitc Policy Gradient


<img src="/surabanke/assets/images/ddpg_objectivefunction6.PNG" width = "400">

Q(s0,a0)를 미분해야한다.

#### q function의 미분
Q를 다시 한번 쓰면 아래와 같다. 여기서 인테그랄 속에 gamma*Q와 P가 곱해져 있는데 이것을 미분하면 A'B + AB'형태가 된다.

<img src="/surabanke/assets/images/ddpg_objectivefunction7.PNG" width = "450">

step 1,2,3으로 미분을 설명한다.

step 1
θ -> a -> R 로 속해있기 때문에 chain rule을 적용한다.

<img src="/surabanke/assets/images/q_function_step1.PNG" width = "280">

a'θ0는 R0안에 있던 aθ0를 미분한 것이다.

step 2
AB'형태를 먼저 나열하자

<img src="/surabanke/assets/images/q_function_step2.PNG" width = "400">

step 3
A'B를 chain rule을 적용하지 않은 상태로 써보면

<img src="/surabanke/assets/images/q_function_step3.PNG" width = "300">

step 1,2,3을 나열하면

<img src="/surabanke/assets/images/q_function_step4.PNG" width = "700">


여기서 파란색 a'θ0 은 s1에 있지 않아도 되는 상관없는 action값이므로 앞으로 나온다.

그러면 a'θ0∙∇aθ0 로 묶어서 아래와 같이 전개 된다.

<img src="/surabanke/assets/images/q_function_step5.PNG" width = "680">

초록색으로 표시한 부분을 보면 Q(s0,a0)이다.

<img src="/surabanke/assets/images/q_function_step6.PNG" width = "500">

Q0를 미분하기 위해선 Q1의 미분값이 필요하다.
그리고 Q1을 미분하기 위해선 Q2의 미분값이 필요하다.
계속 그 다음 step Q function의 미분을 필요로 한다. 아래의 그림과 같이 표현했다.

<img src="/surabanke/assets/images/q_function_step7.PNG" width = "600">

이것을 하나의 식으로 나열해보면..

<img src="/surabanke/assets/images/ddpg.PNG" width = "700">


(아래의 식은 시그마를 써서 표현한것일 뿐 위와 같다.)

<img src="/surabanke/assets/images/ddpg_final.PNG" width = "800">

## DDPG의 특징
DDPG는 off policy다.
target policy와 behaviour policy가 달라야 off policy인데 ddpg는 target policy가 아예 없고(아마 action을 구하기 위한 학습이기때문에 목표로하는 target policy가 없는 게 아닐까) behaviour policy는 policy를 업데이트 하는데 사용되지 않는다고 한다.
어쨋든 이러한 이유로 off policy인 ddpg는 experience replay buffer를 사용하여 correlation을 낮추는 효과를 볼 수 있다.


특징은 앞서 말했던 off policy  parameterized action, action을 한다음 즉시 보상 하는 것과
DQN에서 사용하던 target network, experience replay 기법을 사용한다는 점이다.



## DDPG 알고리즘

<img src="/surabanke/assets/images/ddpg_actor_critic.PNG" width = "540">

논문에서의 알고리즘을 보면




<img src="/surabanke/assets/images/ddpg_algorithm.PNG" width = "900">




샘플이 순차적으로 쌓이는 강화학습환경에서 데이터들끼리의 iid하지 않은데 ddpg는 이를 experience replay로 해결한다.
이것을 uniform random하게 골라서 mini batch로 가져오면 실시간으로
학습하는 것보다 좋은 효과를 얻는다.

critic네트워크는 loss를 최소화하는 쪽으로 업데이트 된다. 여기서 loss는 TD error의 MSE(Mean Square Error)를 사용한다.

<img src="/surabanke/assets/images/ddpg_critic_loss.PNG" width = "350">

위 식은 -E[TD Error]를 표현한 것인데 minimize(-E[TD Error])는 결국 maximize(E[TD error])이고 Q value를 크게 받는 쪽으로 학습을 한다.
