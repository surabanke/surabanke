---
title : "Dynamic Programming"

date : 2020-10-27

categories : rl
---

## Model Based Planning, Dynamic Programming

MDP를 알때 모델의 정보를 이용하여 좋은 policy를 찾는 것을 planning이라고 한다.
동적 : 시간에 따라 변하는 대상을
프로그래밍 : 여러개의 프로세스로 나누어 planning

### Prediction 과 Control

#### Prediction : Estimate the value function

Input : MDP <S,A,P,R,γ> and policy 𝝿 or MRP <S,P,R,γ>

Output : value function(state or action-state function)

기존 policy의 evaluation만 한다.

#### Control : Optimise the value function and find well-made policy

Input: MDP <S,A,P,R,γ>

Output : optimal value function and optimal policy

policy evaluate + improve까지 한다.


- Policy iteration

evaluate과 imporve를 반복하며 최적 policy를 찾는다.
처음 policy를 초기화하고 평가단계에서 policy에 대해 value를 구한다.
policy imporve단계에선 새로운 policy로 업데이트를 하는데 새로운 policy는 앞에서 구해놓은 value function greedy하게 해준값이다.
value function은 bellman expectation equation으로 나타낸다.

\[
𝝿'(s) = argmax q(s,a)
\]

이 절차를 반복 하다보면 optimal policy에 수렴하게 된다.

- Value iteration

policy iteration은 매 iter마다 policy evaluation을 한다.
하지만 이 value iteration은 최적의 policy가 만들어내는 최적 value만을 찾는다.

value function은 bellman optimal euqation으로 한다.
