---
title : "매트랩을 이용한 Local Path planning"

date : 2020-12-08

categories : study
---


## Driving Scenario Designer

Matlab의 Driving Scenario Designer를 이용하여 자율차 시뮬레이터 환경을 만들고
전역 목적지까지 가기 위해 local 목적지를 정한 후 이동하는 강화학습 에이전트구현해 보았다.

### 시뮬레이션 환경 구현

<img src = "/surabanke/assets/images/matlab1.png" width = "200">

<img src = "/surabanke/assets/images/matlab2.png" width = "200">

학습 차량 이외의 차량은 주행경로와 속도를 지정하여 환경을 구현하였다.
레이더를 이용하여 주변 장애물들의 위치를 파악할 수 있다.


<img src = "/surabanke/assets/images/matlab3.png" width = "200">

<img src = "/surabanke/assets/images/matlab4.png" width = "200">

자율차량(에이전트)은 가야할 각속도와 속도, 현재 위치 등을 학습 정보로 사용한다.

### MDP 정의

<img src = "/surabanke/assets/images/matlab5.png" width = "200">

Observation은 현재 좌표, 속도, local target좌표, beamID, 현재 차량과 local target distance, beam의 각도로 설정했다.
action은 steer를 45도씩 -450도에서 450도까지 acceleraion은 0에서 100으로 설정했다.
액셀이 내가 생각하는 차량과 같은 단위인지 모르겠다.

reward는 간단하게, 현재위치가 로컬타겟 위치와 같으면 10점
다르면 -0.01을 두 지점 사이의 거리에 곱해서 패널티를 주었고, 장애물과 거리가 3.5m 이내(차 전장길이보다 좀 작게 설정)로 들어온다면 이것은 충돌로 간주했다.
장외는 무조건 -10점

근데 ego car는 중점이 차량의 후반부에 잡혀있었다.
그래서 아래처럼 학습결과가 나왔다.

<img src = "/surabanke/assets/images/matlab6.png" width = "400">

## 전체 시뮬레이터 구성
<img src = "/surabanke/assets/images/matlab7.png" width = "400">

## 문제점

1. MDP 정의
에이전트의 observation으로 레이더에 잡힌 상대적인 위치와 현재 차량위치 등의 수치 데이터를 사용했지만 모델링을 할 때 CNN을 사용함

리워드를 계산하는 값에 들어가는 로컬 타겟의 정의 식은 전역좌표와 차량과 가까운 장애물의 중간 지점으로 차량이 정상 운행하도록 하는데 충분하지 않음

2. 시뮬레이션 문제
Driving Scenario Designer는 애초에 차량 센서의 작동을 시뮬레이션 하는 것과 자율주행 시스템을 테스트 하기 위한 주행 시나리오 설계가 주요 기능이고
obstacle의 좌표와 겹치면 작동하지 않으므로 충돌 패널티 학습이 어려웠다.
이를 위한 환경을 바꾸거나 obstacle을 임의의 좌표로 주는 방법으로 하면 될것같다.