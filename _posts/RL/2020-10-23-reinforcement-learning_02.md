---
title : "Bellman equation"

date : 2020-10-27

categories : RL
---

## Bellman equation


강화 학습 에이전트는 policy에 맞춰서 기대보상값을 예측하며 그 policy를 평가한다.
이 policy evaluate 하는 과정에서는 bellman expectation equation을 사용한다.
하지만 policy를 evaluation 하는 것만으로 좋은 policy를 얻을 수 없다.
더 많은 보상을 얻게 되는 쪽으로 policy를 업데이트 하기위해선
policy evaluate + policy improve를 해주는 policy control 이 필요하다.
policy control은 MDP가 주어질때 Optimal policy를 찾는 것으로 Bellman Optimal equation으로 설명할 수 있다.

policy를 포함한 value function과 action-value function을 bellman expectation equation으로 표현하면 아래와 같다.


<img src = "/surabanke/assets/images/state_value_2.png" width = "1200">


<img src = "/surabanke/assets/images/state_action_value2.png" width = "1200">


bellman optimal equation은 아래와 같다.
최적방정식에선 s에서 a를 할 확률인 π(a|s)가 사라진다. 액션을 확률적으로 선택하는 것이아니라 가장 좋은 결과를 가져오는 액션을 선택하기 때문에 max를 사용하게된다.
expectation방정식에 max를 한 것과같다.

<img src = "/surabanke/assets/images/optimal.png" width = "1200">
