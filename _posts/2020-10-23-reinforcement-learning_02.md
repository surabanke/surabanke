---
title : "MarkovDecisionProcess"

date : 2020-10-23

categories : 강화학습
---

*Prediction 문제와 Control 문제

강화 학습 에이전트는 policy에 맞춰서 기대보상값을 예측하며 그 policy를 평가한다.
이 policy evaluate 하는 과정에서는 bellman expectation equation을 사용한다
하지만 policy를 evaluation 하는 것만으로 좋은 policy를 얻을 수 없다.
더 많은 보상을 얻게 되는 쪽으로 policy를 업데이트 하기위해선
policy evaluate + policy control 이 필요하다.
policy control은 MDP가 주어질때 Optimal policy를 찾는 것으로 Bellman Optimal equation을 사용하여
