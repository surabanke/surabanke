---
title : "Optimizers"

date : 2020-12-18

categories : DL
---

최적화 기법들에대해 노트에 적어놨던거

###SGD (Stochastic Gradient Descent)

<img src = "/surabanke/assets/images/SGD.png" width = "180">

L은 Loss 로 gradient는 Loss를 Weight로 편미분한 것이다. 여기서 에타는 learning rate

###Momentum

기울기가 변하지 않는 방향으로 일정하게 가속하는 물리법칙을 적용 비등방성 함수에서 움직임 최소화, oscillation 상황에서 관성을 이용하여 불필요한 운동하지 않음.

v <- β*v - α(∂L/∂w)

β는 0.9의 소수로 기울기가 0인 상황에서 v(velocity)를 서서히 감소시키는 역할이다. 물리적인 면에선 공기저항 or 지면 마찰에 해당함

w <- w - v

###Neterov Momentum (a.k.a NAG, Nesterov Accelerated Gradient)

모멘텀 옵티마이저의 변종으로 기본 모멘텀보다 빠르다. 기본 아이디어는 현재 위치가 아니라 모멘텀 방향으로 조금 앞서서 gradient를 계산하는 것이다.

모멘텀에선 w에대해 Loss를 편미분, 네스테로프 모멘텀은 w-βv에대해 Loss를 편미분

###AdaGrad(Adaptive Gradient descent)
adaptive learning rate:
일반 SGD에 learning rate decay를 추가 학습 초반과 후반의 갱신속도 차이를 둠  : global minimum으로 더 빠르게 가며 hyperparameter인 를 덜 tuning해도 된다는 장점이 있다.

h <- h + (∂L/∂w).^2

w <- w - α*(1/sqrt(h))*(∂L/∂w)

h에  ε을 더해서 1/sqrt(h+ε)로 사용하기도 한다. h가 0에 가까워서 전체값이 너무 커지는 것을 방지하기 위함이다. (h는 1e-4 ~ 1e-8정도로)


###RMS Prop

AdaGrad 의 h는 계속 (∂L/∂w)를 제곱하여 갱신되기에 점점 커지고 나중엔 1/sqrt(h)은 갱신될때 0이 되어 전혀 갱신되지 못함 -> 이 문제를 개선한것이  RMS Prop이다.
RMSProp은 EMA(Exponential Moving Average, 지수이동평균)로 과거 기울기 반영 규모를 기하급수적으로 감소시킴

--내 생각--
아까 AdaGrad에서  기울기가가 너무 커지는 것을 방지하기 위해(그럼 h는 0에 가까워짐) 아까 ε을 더해서 1/sqrt(h+ε)로 쓰기도 한다고 했는데(h가 0일때 ε값을 더해서 무한으로 가는것을 막자) 그것은 임시방편이었는지 h가 업데이트 될 때 기울기 자체를 줄이는 방법을 고안한 것이  RMSProp 인거 같다.

h <-  γh + (1- γ)((∂L/∂w).^2)

γ(gamma)는 decay rate

w <- w - α*(1/sqrt(h))*(∂L/∂w)


###Adam(Adaptive moment estimation)

Momentum + RMSProp 으로 부드러운 움직임과 매개변수 갱신도조정 특성을 다 포함한 최적화기법

1,  v = γv + (1- γ)(∂L/∂w) moment 처럼 gradient 곱

2,  h = γh + (1- γ)((∂L/∂w).^2) rms prop처럼 gradient  제곱

3,  v' h'를 계산 (bias correction)

4,  w <- w - α*(v/sqrt(h+ε))


3번에서 왜 bias correction 을 하는지?

이는 학습 초반에 v0와 h0가 0으로 가는 문제를 보완하기 위해 추가한것이다.
수식의 1,2번과정을 보자 v0와 h0값이 0이라면  γ_v와 γ_h 는 1이 되면서 학습 초기에 γ_v와 γ_h의 기여도가 없어지는 문제가 생긴다.


<img src = "/surabanke/assets/images/adam_bias_correction.PNG" width = "600">


v0와 h0가 0이라고 가정하고 현재시간 t부터 이전시간들의 기울기를 전부 더하면


<img src = "/surabanke/assets/images/Adam_1.PNG" width = "300">


 논문 (1)식에서 보듯이 표현할 수 있다. 여기서 β는  γ인듯 하다.

v(t)의 평균은 크데 두 값의 곱으로 표현된다. t에서 기울기인 E[(∂L/∂w)] 와 상수 (1-γ^t)이다.

이렇게 얻은 (1-γ^t)로 v(t)를 얻을 수 있고 이는 v(t)값을 증폭시킨다.

v'와 h'를 만드는 이러한 과정을  bias correction이라고 하고 초기값  0에 가까워지는 v와h를 기댓값을 얻어내 구한 (1-γ^t)으로 나눔으로써 보정하는 것을 말한다.

이 v'와 h'를 가지고 가중치를 업데이트 하는 것이 Adam optimizer 이다.

논문에서는 learning rate은 1e-3, v에 사용된 γ는 0.9 h에 사용된 γ는 0.999 ε는 10^-8값으로 명시되어있다.

Nadam은 momentum  대신 NAG를 사용한 adam optimizer임


### Adam, Convergence analysis

Adam은 convergence analysis 단계가 있다. 가장 잘 구해진 J(θ) loss 함수와 비교하는 regret 을 구한다.


<img src = "/surabanke/assets/images/Adam_converge_analysis.PNG" width = "300">


여기서 f(θ*)이 best loss다.

'Sum of all the previous difference between online prediction and the best fixed one.'


atom 편집기를 쓰고 있는데 수식쓰는게 너무 귀찮다.
코드형식으로 쓰려니까 불편하다..
