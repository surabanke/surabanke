---
title : "Goal-Oriented Next Best Activity Recommendation using Reinforcement Learning 논문리뷰"

date : 2023-08-09

categories : rl
---
## 비즈니스 상황에서 여러가지 조건을 만족하는 최적의 action 고르기

ongoing case에 대해 행동을 추천하려면 기본  비즈니스 프로세스를 준수해야하며 time and outcome을 충족해야한다.

이 논문은 딥러닝으로 가장 좋은 활동과 해당 활동을 할때 KPI 달성률이 어느정도 되는지를 예측하고 이를 바탕으로 강화학습을 써서 하나 이상의 목표를 충족할 가능성이 있는 일련의 활동을 탐색한다.

즉, 순차적으로 진행중인 사례에서 다음 이벤트에 대한 결정을 내려야하는데 가장 KPI를 잘 완수해내는 선택을 한다.
이는 어려운 문제이다. 왜냐하면 대부분 비즈니스 영역에서 최적화를 원하는 KPI인 quality, time, cost등이 서로 상충하기 때문이다.

예를 들어 옷을 만드는데도 질을 높이려면 옷 한벌을 만드는 시간과 공장 가동 비용이 증가한다.
반대로 시간과 공장 가동 비용을 줄이려면 옷의 질이 떨어진다.

또 다른 예로 금전 대출 신청 프로세스를 생각해보면
제한된 조치로 대출 시퀀스를 완료하고 지정된 시간 내에 대출 신청을 승인하면 신청자의 신용점수를 확인하지 않고 대출이 승인될 수 있으므로 대출가능조건을 충족하지 못한 신청자일 수 있다. 이는 대출 승인 처리시간과 모순된다.
RL은 이러한 문제에 적합하다.

이런 문제를 논문에선 goal-oriented and conformant activity sequence prediction한다. (=KPI를 잘 지키는 연속 행동 예측)
강화학습을 사용하지 않은 이전 연구에선 undesired outcome(원치 않은 지출)이 일어날 가능성을 추정하는 확률적 분류기와 그 확률이 임계값을 넘어서면 알람이 울리게 두 단계로 구성한 Intervention based recommend가 있다. 이때 최적의 임계값은 각 도메인 비즈니스 프로세스의 개입비용으로 한다.

**개입비용이란?**

예를 들어 주문-배송 프로세스에서 고객이 주소를 제대로 기입하지 않을 시에 수동적으로 기다리지 않고 누락된 정보를 얻기 위해 고객에세 전화를 걸을때 물품을 생산하는데 드는 노동임금, 원료비 등 직접비용 이외에 소용되는 제반비용으로서 명확하게 구분할 수 없는 비용을 말한다.

기존 데이터에 대해 학습된 랜덤 포레스트를 사용하여 케이스의 **현재 상태만을** 고려해 개입이 주기시간 감소에 미치는 효과를 추정한다.

그러나 이런 방법은 바이너리적이며(개입해? 말아?) 최적의 다음 단계 활동을 어떻게 해야하는지 권장하는 기능은 없다.
개입이 적용된 후의 프로세스에서 취해져야 할 최적의 단계는 추천하지 않는다.

또 다른 관련 연구로 suffix recommendations가 있다.
딥러닝 모델로 다음단계 및 KPI를 예측하도록 학습하는것이다.
개입이 진행중인 케이스는 suffix(다음으로 해야 할 행동) 및 KPI예측값이 제공된다.
후보 suffix들이 기존 데이터(그 상태에서 할 수 있는 여러 행동들)에서 KNN으로 예측된 값과 비슷한 것들을 추려내어 선택된다.
그 후 이 후보군은 비즈니스 시뮬레이션 모델로 들어가 리스크가 가장 적은 최적의 suffix를 추천한다.
그러나 후보군 중에 KPI의 조건을 만족하지 못하는 후보들로만 구성된다면?
그러한 문제에대해 답을 내놓은 것이 바로 이논문이다.

### 본문

이 논문에서는 DFG(Directly-Follows-Graph)라는 비즈니스 프로세스의 그래프를 사용한다.

<img src="/surabanke/assets/images/DFG.png" width = "600">

진행중인 일에 대해 행동을 추천하려면 기본  비즈니스 프로세스를 준수해야하며 time and outcome을 충족해야한다.
KPI prediction model영역의 DL Model M이 가장 좋은 활동과 해당 활동을 할때 KPI 달성률이 어느정도 되는지를 예측하고
이벤트로그에서 가져온 state와 Model M에서 예측한 KPI달성률 그리고 이 시점에 선택 할 수 있는 action을 DFG를 보고 가져와서 강화학습에 사용한다.
강화학습은 위의 RL Agent영역에서 보다싶이 Maskable PPO를 사용하며 하나 이상의 목표를 충족할 가능성이 있는 일련의 활동을 탐색한다.
즉, 진행중인 사례에서 다음 이벤트에 대한 결정을 내려야하는데 각 선택이 KPI(완료시간)과 궁극적 목표(완료시간 내에 완료했는지)에 영향을 미칠때 가장 KPI를 잘 완수해내는 선택을 하게한다.

용어

이벤트 로그(데이터 저장소) 안에 trace라는게 있다.
trace: 시간 흐름에 따른 이벤트들의 집합 trace =  {e1,e2,e3,…em}
event e는 trace의 구성요소로  e = {trace_id, activity a, timestamp ts}로 이루어진다.

<img src="/surabanke/assets/images/trace.png" width = "600">

### 리워드

<img src="/surabanke/assets/images/bestactreward.png" width = "400">

KPI 끼리 상충될 수 있어서 goal 1이 시간이고 goal2가 비용이면 ground truth trace의 action에 따른 비용 분포도(ground truth distribution of process outcome)를 구하여 action이 그 값이랑 다르면 -0.5 맞으면 +0.5를 주는 balancing reward를 도입하였다.

### 예제
예제 코드를 보면

문제 정의 : 비즈니스 KPI를 고려하여 강화학습으로 최적의 행동을 추천
문제의 목적 : 독일 금융회사의 대출 신청 프로세스가 진행 중 일때 시간과 비용을 고려하여 다음 진행 해야하는 행동을 추천

**STATE**

이전t-1에서 예측된 Action과 딥러닝에서 예측한 value로 구성 St = (at-1, Vt-1)

**(이전 Action값, KPI value값)**

**ACTION**

논문의 데이터셋 4개에는 각각의 이벤트 데이터를 토대로 각 상황의 관계를 나타내는 그래프로 만들어져 있다.(process mining tool을 사용하였다고 한다.)
아래 그래프와 같이 각 이벤트의 관계를 나타낸 그래프를 만듬

<img src="/surabanke/assets/images/DFG2.png" width = "300">

코드에 그래프 관계가 하드코딩 되어있다.

<img src="/surabanke/assets/images/DFGcode.png" width = "600">

대출 신청 프로세스 그래프 관계에 따라서
현재 노드가 3일때 가능한 action들은 3, 5, 0이 있고
노드 3을 고르면  (대출 신청을 위한 정보 작성)을 계속 진행
노드 5를 고르면 다음 노드인(대출 신청 한 후 연락하기)를 진행
노드 0을 고르면 대출 신청 프로세스 끝 (그래프 상에서 빨간 정지버튼)
이때 가장 최적인 action을 고른다.
그래프의 노드들은 총 6개로 Initial action또한 [0,1,2,3,4,5,6]

**위 그림의 빨간 숫자 기준의 action**

1. **수신 리드 수정**
2. **사기 행위**
3. **신청서 정보 작성**
4. **신청서에 부족한 정보 추가하기 위해 전화하기**
5. **제안서를 보낸 후 전화하기**
6. **신청서 평가**

그러나 위 내용 처럼 비즈니스 프로세스 그래프에 따라 action masking되며 각 노드에 따라 선택 할 수 있는 action이 달라진다.

**REWARD**

비즈니스 프로세스가 종료되기까지 특정 행동이 좋은 선택이었는지 나쁜 선택이었는지 모르기 때문에 즉시 보상 값은 0으로 준다.
그리고 event가 끝났을때만 누적 리워드를 계산

- Mean Absolute Error(MAE) : KPI prediction model(딥러닝)의 오차
- KPI prediction model에서 예측한 누적 KPI value의 합 : G(trace)
- threshold : 시간 최적화가 되었는지에 대한 임계값으로 하이퍼파라미터로 설정

G(trace)와  MAE 합이 threshold보다 작으면 goal을 만족한다고 생각함
GS(Goal Satisfaction) = G(trace) + MAE =< threshold

Reward_1 식 : **2x(G(trace)-threshold)/threhold**

첫번째 KPI가 시간이고 두번째 KPI는 비용이므로 비용에 대한 최적화가 되었는지는
ground truth 데이터의 히스토리 action리스트를 보고 현재 이벤트의 action 리스트와 비교하여 맞으면 +0.5 틀리면 -0.5를 한다.
action 리스트를 비교하면서 누적된 리워드 값을 앞의 Reward_1식과 더하여 최종 리워드 값이 된다.

**Test 결과**

KPI (primary goal:시간, secondary goal: 비용)

episode, 1

reward 0.2510

- 비용에대한 성능
    1. accuracy_last 0.382 (마지막 이벤트의 정확도)
    2. accuracy_last2 0.3212443 (마지막 2개 이벤트에 대한 정확도)
    3. accuracy_last3 0.2922 (마지막 3개 이벤트에 대한 정확도)
- 시간에 대한 성능

    goal stisfaction : 0.99 (goal 만족)

    goal violation : 0.00023299 (goal 불만족)

    goal violation_turned_goal stisfaction : 1.0( gv인 GroundTruth 이벤트를 학습한 모델이 gs로 바꿀수있는 확률)

    goal stisfaction_turned_goal violation :  0.000313 ( gs인 GT 이벤트를 학습한 모델이 gv로 바꿀수있는 확률)


시간과 비용이 trade-off이기 때문에 비용 성능이 높으면 시간 성능이 낮아지는 경향이 있다.
