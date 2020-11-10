---
title : "Model free Learning_1"

date : 2020-11-04

categories : 강화학습
---
## MDP를 모를때 Model free Prediction

bellman equation을 사용한 dynamic programming방식으로 state value나 action value 계산하고 그것을 greedy 하게 해준값을 구하여
policy를 업데이트 하면되지만 MDP를 알 수 없는 상태에서 P(s-> s'로 넘어가는 trainsition prob)이나 reward를 모르기때문에 이 방식으로 구할 수 없다. MDP를 모르는 상황을 model free라고 하고 이 상태에서 policy를 업데이트하는 과정없이 기존의 것을 사용하여 value를 evaluate하는 과정을 model free prediction이라고 한다.

model free상태에서 value function값을 구하는 알고리즘으로는 montecarlo와 temporal-difference 두가지가 있다.



#### Monte Carlo Prediction

*V(s) = total_return / N(s)*

여기서 V(s)는 state s에서의 총 리워드의 기댓값이며 에피소드중 받은 총 리턴값을 에피소드 중 s를 방문한 횟수N(s)로 나누어 구한다.
여러번 반복하여 샘플 갯수가 많아질 수록 더 정확해진다.(Law of Large numbers)

s0.a0,r0,s1,a1,r1,s2,a2,r2,...,sT-1,aT-1,rT-1,sT -> 에이전트가 학습하면서 받는 데이터

모든 에피소드가 다끝나고 평균을 내는것 보다. 한 에피소드가 끝날때마다 값을 업데이트하는 방법도 있다.

total return = G

α = update parameter

N(s) = state s를 방문할 때마다 +1

*V(s) <- V(s) + 1/N(s)(G - V(s))*


<img src = "/surabanke/assets/images/MCbackup.png" width = "900">


total return G와 현재 reward 인 V(s)의 차이를 최신 리워드와 더해준다.

 1/N(s)를 그냥 α로 쓰고 learn rate 처럼 고정시켜도 좋다.

 α가 클수록 G 즉 모든 리워드들을 합친 총 리턴값이 작아지고(forget old episodes) 새로운 값(V(s))을 더 많이 적용한다.

 딥러닝에서 gradient update를 하여 global minimum loss를 찾는것처럼 매번 최종 목표인 G에 가까워지는 V(s)를 볼 수 있다.
 G보다 V(s)가 작으면  α값이 곱해진 양수가 되어 V(s)를 크게하고 G보다 크면 음수가되어 V(s)값을 낮춰 total return값에 가까워지게 한다.

#### Temporal-Difference Prediction

<img src = "/surabanke/assets/images/TDbackup.png" width = "900">

*(V(s) <- V(s) +  α(Rt+1 + γV(St+1) - V(St))*

다음 state t+1 의 return *V(St+1)*을 추측 그것을 t일때 value에 반영.

TD target 쪽으로 V(s)를 업데이트 한다. 즉 한 step차이인 다음 step값과 현재값의 차이를 update한다.



#### TD와 MC의 큰 차이점


TD can learn before knowing the final outcome.

MC must wait until end of episode before return is known.

TD can learn from incomplete sequences -> step별로  동작

MC only learn from complete sequences. -> episode별로 동작

MC는 높은 variance zero bias 각 episode마다 update
TD는 낮은 variance some bias(편향) step마다 update


#### λ(lambda) return

TD를 한 step마다 update하는게아닌 n-step마다 update 할 수 있다.
TD(∞)는 에피소드가 끝나야 알 수 있기 때문에 montecarlo와 같다.
MC로 갈수록 사용하는 weight가 줄어든다.

Forward-view TD(λ)

<img src = "/surabanke/assets/images/forward-view.png" width = "900">

geometric mean을 사용하여 평균을 낸다. 다 더하면 1이 나온다.
Forward-view는 Gt(λ)를 계산하기 위해 미래의 return값을 본다.
MC와 같이 에피소드가 끝나야만 알 수 있어서 TD가 갖는 장점을 잃는다.


Backward-view TD(λ)

<img src = "/surabanke/assets/images/backward-view.png" width = "850">

Eligibilty traces

Frequency heuristic : 가장 많이 일어난 일에 책임을 묻는다.
Recency heuristic : 가장 최근에 일어난 일에 책임을 묻는다.

-keep an eligibility trace E[s] for every States

-Update value V(s) for every states.

-TD error는 δ = (Rt+1 + γV(St+1) - V(St))

*V(s) <- V(s) + α*δ*E[s]*

->과거에 지나온 모든 state의 eligibility trace를 기억해 뒀다가 그만큼 update한다.


## Model Free Control

MDP를 모를때 value를 구하여 policy evaluate 할 뿐 아니라 policy를 update한다.

-On policy : π, π

-Off policy : π, μ

학습할때 사용하는 policy와 experience할때 사용하는 policy가 같으면 On 다르면 Off policy라고 한다.

#### On policy Monte Carlo Control

evaluation은 Monte Carlo policy evaluation으로 policy improvement는 e-greedy를 사용한다.
(epsion의 확률로 random한 action을 한다. exploitation, exploration 다 할 수 있다.)

value function은 action state value를 사용한다. 왜냐면 value function은 여러 action중 어느것을 선택할지 확률을 알아야하는데 MDP를 모르는 상황에선 구할 수 없다.
반면 action state value 는 action이 주어지고 action으로 인해 갈 수 있는 모든 다음 state의 transition probability까지 알 수 있기 때문에 사용가능하다.

policy를 improve하는 단계에서는 e-greedy를 사용한다.
이때 GLIE(Greedy in the Limit with Infinite Exploration)를 만족해야하므로 epsilon greedy e를 1/k로 점차 줄어들게 한다.

<img src = "/surabanke/assets/images/GLIE.png" width = "850">

GLIE 1, 많은 횟수에 걸쳐 exoploration이 진행되면 N -> ∞
GLIE 2, 결국 converge to greedy.

<img src = "/surabanke/assets/images/MC_GLIE_Predict.png" width = "500">

<img src = "/surabanke/assets/images/MC_GLIE_Control.png" width = "500">

이것을 GLIE Montecarlo control이라고 한다.


#### On Policy TD Control a.k.a SARSA

<img src = "/surabanke/assets/images/Onpolicy_SARSA.png" width = "500">

SARSA는 GLIE와 Robbins Monro 조건을 따른다.

<img src = "/surabanke/assets/images/Robbins-Monro.png" width = "200">
