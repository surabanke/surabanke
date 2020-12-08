---
title : "TRPO"

date : 2020-12-0

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
그러나 어느 정도 범위내에서 이 근사치가 허용될 수 있는지 알수 없다.(어느 정도 변해야 objective function의 improve를 보장하는지 알 수 없다.)
그 trust region을 찾기 위하여 kakade&langford는 conservative policy itertation이라는 policy update방법론을 제시한다.

<img src = "/surabanke/assets/images/mixture_policy.png" width = "300">

하지만 보통 policy를 섞어서 학습에 사용하진 않는다고 한다. 그래서 고안한 다른 방법이

<img src = "/surabanke/assets/images/lowerbound.png" width = "300">

위의 식이다.
()이 식의 증명식도 논문 뒤쪽에 있는데 보지않았다.)
어쨋든 이 식은 우측이 좌측식의 improve를 보장하고 있다는 것을 보장하고 있다.
