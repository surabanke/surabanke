---
title : "MarkovDecisionProcess"

date : 2020-10-26

categories : 강화학습
---

-팡요랩과 실버강의 바배강 책을 참고하여 쓴 필기를 정리함

**Markov Process
MDP는 강화학습을 할때의 환경을 설명할때 자주 언급된다.
미리 정의된 전이확률(transition probability)를 따라서 상태가 변화하는 프로세스이며 전체 프로세스의 transition probability 의 합은 1이다.

마르코프 프로세스의 모든 state는 Markov property를 따른다.
Markov property 현재의 상태가 주어지기만 하면 미래의 상태를 결정하는데 충분하다는 것이다
즉, 과거의 상태들이 어떻든간에 미래의 상태에 영향을 줄 수 없다.
식은 조건부 확률을 사용한 전이확률로 정의된다.

딥마인드사에서 비디오게임을 학습시킬때 t(현재)뿐만 아니라 t-2,t-1등 과거의 이미지도 엮어서 학습시킨 것도 좀 더 마르코프한 상태를 만들기 위함이다.

**Markov Reward Process
Markov Reward Process 는 transition prob에 reward와 discount factor 개념이 더해진 것이다.

*MRP = (S,P,R,γ)
이 중 discount factor인 감마는 0에서 1사이 숫자이며 학습에서 미래에 얻을 보상보다 당장에 얻는 보상이 얼마나 더 중요한가를 나타내는 파라미터다.

**Markov Decision Process
MP와 MRP는 다음 상태로 갈 전이확률이 정해져있다. 미리 정해진 확률에 따라 상태도 정해진다.
의사결정을 하기위해서는 action이 필요하다.이 의사결정 변수인 action값으로 인해 다음상태가 달라지고 그에따른 보상을 받는다.
*MDP = (S,A,P,R,γ)
[<img src="./assets/images/change_transition.png" width = "50">]
