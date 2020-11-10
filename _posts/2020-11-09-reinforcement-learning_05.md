---
title : "Model free Learning_2"

date : 2020-11-04

categories : 강화학습
---

## Model free control Off-policy편

이전 편에선 policy를 업데이트하고 action을 정하는 policy가 같은 On policy까지 작성하였다.

off policy는 behaviour policy와  target policy 을 달리하는 방법이다.

off policy의 장점은 사람이나 다른 에이전트로부터 경험을 배울 수 있다는 것이고
exploration을 계속하면서 optimal 한 policy를 찾을 수 있다는 것이다.



#### Importance Sampling

Importance sampling은 통계학에서 알고자 하는 분포와 다른 분포에서 샘플을 가져와서 특정분포의 특성을 추정하는 기법이라고 한다.

원하는 확률분포에서 샘플링이 가능하지 않을 때 다른 분포를 통해 이를 추정한다.

<img src = "/surabanke/assets/images/ImportanceSampling.png" width = "500">

위는 확률분포 P를 따른 확률변수함수 f(x)의 기댓값을 다른 확률분포인 Q를 이용하여 나타낸 식이다.
(P/Q는  importance weight이라고 한다.)

#### Off-policy Monte Carlo Control
π, μ
강화학습에선 이 확률분포들을 policy로 바꾸어 사용한다. π는 evaluate policy고 μ는 behaviour policy이다.
monte carlo의 update target은 Gt - Vt(S)이다. Gt는 behaviour policy를 따를때 reward에 discount factor를 적용한것의 총합이기때문에 이것을 target policy(π) 에대한 return값으로 변형해야한다.

그러므로 Gt에 π/μ(importance weight)를 곱한다. (π에대한 확률분포로 만들어주는것과 같다.)

Monte carlo evaluate기법을 하면  (π/μ, importance weight)을 계속 곱해주어야 한다. 발산할 수도 있고 너무 작으면 0으로 수렴할 수도 있기때문에 TD를 사용한다.

#### Off-policy TD Prediction

<img src = "/surabanke/assets/images/TDControl_offpolicy.png" width = "400">



Off policy는 바로 앞step 하나에 대해서만 importance weight를 곱해주면 된다.




#### Q Learning (Off-policy TD Control)

Q Learning은 importance sampling을 사용하지 않아도 된다.
현재 state에서 behaviour policy를  ε-greedy로 따라 action을 선택하고 q-function을 이용하여 update 하고 이때 TD target 다음 state s'에서 a'는  target polilcy π 를 greedy로하여 action을 선택한다.
이렇게 하면 직접 경험 한 것이 아니라 다른곳에서의 경험을 가져다 사용할 수 있다.

Q Learning은 On policy SARSA 알고리즘(벨만 기대방정식)과 다르게 벨만 최적방정식을 사용한다.

<img src = "/surabanke/assets/images/Q-learning.png" width = "400">
