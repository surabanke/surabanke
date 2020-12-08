---
title : "TRPO"

date : 2020-12-08

categories : 강화학습
---


## Trust Region Policy Optimization

TRPO 어렵다

TRPO는 rl->model free learning->policy based RL에 있는 알고리즘이다.
이전의 stochastic policy optimization 의 문제점을 보면

policy gradient 로 policy를 계속 갱신하면 이전 policy로 얻은 data이용하기가 어렵다
-> sample efficiency가 안좋음

distance in parameter space != distance in policy space

parameter θ 에서의 작은 스텝변화가 policy π 에선 크게 작용 될 수 있다.

 -> policy가  조금씩 변하는 stepsize를 찾아야함

trust region policy optimization은 이런 문제를 보안한 안정성있는 policy update를 위한 알고리즘이다.
아직 이해가 덜 되었는데 후에 이해가 정확히되면 수정하려고한다.
읽기 전엔 블로그에 논문리뷰를 쓰려고 했는데 내가 이해하기에 너무어려웠다.

이 글은 팡요랩선생님들이 리뷰한 영상, RLkorea블로그, 원 논문을 읽고 가까스로 이해한것들을 적어놓은 것이다.

Abstract를 보면 여러 게임에서 하이퍼파라미터 변경없이 범용적으로 사용 할 수 있다고 나와있다.

Introduction에서는 이 알고리즘의 surrogate objective function을 minimize하는게 policy를 improve한다는 것에대해 증명하고 이 함수의 근사치를 만들어서 학습을 시키는데 이것을 trust region이라고 부르는 이유는 근사함수가 실제 함수의 역할을 할 수 있는 구간이 있고 그 범위를 넘어가면 안되기 때문인것같다.
논문 뒷장을 보면 이 범위를 제약조건으로 달아서 constraint optimization알고리즘이된다.


<img src = "/surabanke/assets/images/objective_function.png" width = "300">


이 식은 policy를 직접 update하는것으로 학습을 진행 할 때 expected  discounted reward를 구하는 것이다. 여기서 에타는 policy π의 성능지표로 objective function으로 생각해도 될거같다.

그리고 kakade & Langford는 old policy에서 new policy로 업데이트 할때

에타(now policy) = 에타(old policy) + new policy로 trjectory를 뽑을때 expected reward function을 구하는데 그 reward function으로 old policy에서 구한 adventage finction을 사용한

<img src = "/surabanke/assets/images/kakade_langford.png" width = "300">

식을 이용해서 new policy를 구할 수 있다고 주장했다.

논문에서는 뒤에 A.proof of policy Improvement Bound라는 제목으로 위의 식을 증명해주는 식을 기재했다.
(여기선 생략 인쇄한 논문참고)

이 식의 E를 state space와 action space로 나누어 보면 아래와 같은 형태가 된다.

<img src = "/surabanke/assets/images/sumoverstates.png" width = "300">

여기서 P는 새로운 policy로 진행할때 이 state에 에이전트 머무르는 횟수이다.
(새로운 policy를 구하는 과정에 이 policy일때의 visitation frequancy라니 이상하다.)
(P는 transition probability,P(St+1|St)로 인식되어서 그런지 visitation 횟수랑 헷갈린다.)

그리고 모든 state에 대한 discounted visitation frequency를 심볼 ρ(rho)로 표현된다. 그리고 new policy일때 이 state에서 특정 action을 고를 확률을 old policy로 구한 adventage function과 곱하여 준다.

이 식의 뜻은 만일 모든 state s에대하여 adventage가 0보다 크거나 같다면 policy 성능인 에타가 증가한다는 것을 보장한다는 뜻이다.

하지만 이 식에서 계산하기 복잡한 것이 있다.(새로운 policy를 구하는 과정에 이 policy일때의 visitation frequancy라니 이상하다.)아까 언급했던 이것.
new policy에서 로우를 구하는 것이 아니라 기존 policy로 visitation frequency를 구하여 η(π')를 근사하는 L(π')을 정의한다.


<img src = "/surabanke/assets/images/approximate_eta.png" width = "300">


논문의 3번식이다.
그리고 식 (4)는 policy 를 θ로 parameterize했을때 L(π)와 η(π)가 first order로 같으면 둘은 미분해도 같다는 걸 의미하며 L을 improve하는 것이 η를 improve하는 것과 같다는 것을 보여준다.
그러나 어느 정도 범위내에서 이 근사치의 사용이 허용될 수 있는지 알수 없다.
(= 어느 정도 변해야 L(π')근사objective function의 improve를 보장하는지 알 수 없다.)
근사치를 추정 할때 Σ π'(a|s)A(s,a)<0 이 되는 approximation error가 발생하지 않기 위한 trust region을 찾기 위하여 kakade&langford는 conservative policy itertation이라는 policy update방법론을 제시한다.

<img src = "/surabanke/assets/images/mixture_policy.png" width = "300">

여기서 new policy와 old policy를 alpha 와 1-alpha를 사용하여 어느 policy를 얼마나 사용할지 결정한다.
하지만 보통 policy를 섞어서 학습에 사용하진 않는다고 한다. 그래서 고안한 다른 방법이

<img src = "/surabanke/assets/images/lowerbound.png" width = "300">

위의 식이다.
(이 식의 증명식도 논문 뒤쪽에 있는데 보지않았다.)
어쨋든 이 식은 우측의 improve가 좌측식의 improve를 보장하고 있으며 η(π')의 lower bound를 제시한다.
이 중 alapha를 new policy와 old policy의 total variation divergence라고하면 식 (8)이된다.

 <img src = "/surabanke/assets/images/Theorm1.png" width = "300">

total variation divergence는 모든 state에 대해 old policy와 new policy가 어느 state에서 가장 많이 다른가를 그리고 그 state에서의 차이를 나타내는것이지 두 policy를 mixture한 것이 아니다.

TV divergence보다 KL divergence가 더 큰 개념이기때문에 여기에 KL 발산을 사용할 수 있다.

 <img src = "/surabanke/assets/images/KL_div.png" width = "300">

식 (9)에서 L-C*Dkl 부분을 M이라고 정의 할때

 <img src = "/surabanke/assets/images/surrogate_function.png" width = "300">

(10)이 성립되는데 이것으로 M을 극대화하는것은 에타를 극대화하는 것과 같다고 볼 수 있다 여기서 M을 에타의 surrogate function이라고 한다.

완성된 알고리즘을 보면  eta policy의 surrogate function을 argmax하여 성능이 줄어들지 않게 보장한다

 <img src = "/surabanke/assets/images/algorithm1.png" width = "300">

그러나 practical하지 않은 부분이 있다.
###### 1 모든 state에서의 adventage function을 구하는 것은 실질적으로 불가능하고
###### 2 모든 state에서 KL divergence를 구하여 그 중 가장 큰 값을 구해야 하는데 이것도 역시 불가능 함

###### 3 C는 너무 큰 값이다. M(surrogate function)을 최대화하고 싶은데 C 때문에 step size(= update해도 티가 안난다.)가 너무 작아진다.

그래서 챕터4에선 practical 한 알고리즘으로 만드는 과정이 쓰여있다.
policy π -> θ 로 parameterize하고 (1)

 <img src = "/surabanke/assets/images/parameterize.png" width = "300">

그리고 여기서 C는 이론적인 숫자라 실제로 쓸 수 없다. KL divergence는 δ보다 작게하면서 constraint optimization형태로 만든다. (2)
KL divergence를 모든 state에서 구할 수 없으므로 일부를 샘플링하여 그것의 평균을 사용한다.
(sample평균은 모평균의 unbiased estimator이므로 사용가능)

 <img src = "/surabanke/assets/images/constraint_optimization.png" width = "300">

즉 이 식은 논문 챕터 5의 제목과 같다. (Ch.5 Sample-Based Estimation of the Objetivem,L and Consratint,KL div)

이것을 몬테카를로 simulation으로(1 episode끝나고 update) 만들어서 근사치 추정하여 new polict로 update하는 거로 바꾸려면

먼저 expectation 형태로 만들어야 한다.
여기서 η(π')의 근사치인 L(π')를 parameter화 시킨 maximize 식을 다시 가져 온다.

<img src = "/surabanke/assets/images/equation13.png" width = "300">

이 식에서 action부분을 importance sampling을 이용하여 expectation식으로 만들고

<img src = "/surabanke/assets/images/equation14.png" width = "300">

state부분도 s가 ρ에서 sampling됐을때 기댓값을 구하는 식으로 만들어서 (14)식과 같게 만든다.
그리고 A를 Q로 대체한다. (13)과 (14)는 형태만 다를 뿐 같은 식이다. 이렇게 expectation식을 만들면 motecarlo simulation형식으로 학습하고 update하는 것이 가능해진다.(이해 어려움)

그리고 학습 절차로 single path 와 vine이 있는데 이 다음은 다음에 공부하기로..
