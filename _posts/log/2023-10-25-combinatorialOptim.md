---
title : "Combinatorial Optimization with RL"

date : 2023-10-25

categories : log
---

### 조합 최적화

조합 최적화(combinatorial optimization)는 유한한 조합 중에서 최적의 것을 찾아내는 문제를 다룬다.
조합최적화를 위한 접근법을 아래와 같이 나눠보고 강화학습을 이용할때는 언제인지, 어느 방식으로 사용하는지 조사해보았다.

### Heuristics

- SA: (담금질을 하면서 전역최적화를 찾기 때문에 시간이 오래걸림)
- GA: 서치공간을 줄이고 남은 후보군에서 여러가지 조합을 섞어 최종적으로 선택된 조합을 추천
- Branch & Bound :각 노드를 방문할 때 마다, 그 노드가 유망한지의 여부를 결정하기 위해서 한계치(bound)를 계산한다.

한계치가 이전까지 찾은 최고 해답값보다 더 좋으면 그 마디는 살아남고, 그렇지 않으면 더 탐색하지 않는다.

선형계획법은 최적해가 없을 공간까지 가능한 영역을 전부 탐색함, **제약식이 많아도 해결하기 어렵다**.
이 문제를 보완하기 위해 Branch & Bound가 나왔으며 이산적인 문제에 접근 가능하다.

그밖에도 범용적인 휴리스틱 최적화 알고리즘으로
Tabu Search, Ant Colony Optimization, Particles Swarm Optimization 등이 있다.

### Deep learning

각각의 특성이 노드가 되고 그 노드 간의 엣지를 구성한 그래프 형태의 데이터로 구성하여 graph neural net을 많이 사용한다.

<img src = "/surabanke/assets/images/graphNeuralNet.png" width = "500">

딥러닝을 사용하려면 지도학습 방식이므로 데이터셋에 라벨이 있어야 분류가 가능하다.
GCN f(G,theta)에서 그래프의 각 노드가 optimal solution일 확률 맵을 만든다.
완벽히 라벨링된 데이터는 Tree Search의 노드가 되며 내부노드들은 그 과정에서 생성되는 불완전한 라벨링을 나타낸다.
Tree Search에 의해 생성된 완전한 라벨링은 local search를 통해 정제되고 최종optimal solution이 선택된다.

Combinatorial Optimization with graph convnet and guided tree search Paper에서
전체 데이터셋 중 단 5%만 label이 있어도 나머지에 대해 잘 분류한다고 하지만 그럼에도 소수의 라벨 데이터가 필요하다는 말이 된다.
주어진 데이터에 대해 신뢰할 수 있는 검증 라벨이 존재하지 않는 경우에는 아래와 같은 방법을 사용한다.

### Reinforcement learning

**LeNSE**

RL은 search space를 줄이는데만 사용하고 최적화는 휴리스틱을 사용함
LeNSE (NeuralNet LargeScale SearchSpace Combinatorial Optimization)에서는 강화학습으로 subgraph 찾는다.(search space 줄임)
graph pruning이라고 하며 전체 search space에서 찾는것보다 optimal solution을 찾을 확률이 더 크다고한다.

RL(Subgraph navigation policy)학습에 사용할 Dataset을 만들때
(subgraph, y) y라벨링을 해줘야하는데 이때 라벨 랭킹이 높을수록 해당 그래프에 optimal solution이 있을 확률이 높다는 것이다.
y는 heuristic solver(original graph에서의 optimal solution이 들어감)라는 식으로 구한다.


**DRLSA**

<img src = "/surabanke/assets/images/DRLSA.png" width = "800">

SA의 initial solution이 DeepRL의 init state가 된다.
몇 step은 강화학습으로 진행하여 init solution을 찾은 후 simulated annealing을 진행한다.
('they run SA every few steps to explore the region around the current RL solution.')
DeepRL의 state가 SA의 current best solution이 되는것이다 더 좋은 Sbest(best solution)를 찾으면 대체된다.
128개 중 35개의 케이스만 original SA가 성능이 더 좋았고
54개는 DRLSA가 더 좋았다고한다.


**Neural SA(Neural Simulated Annealing)**

위의 DRLSA와 다르게 RL의 optimisable한 요소를 SA와 같이 사용한다.(solution을 share한다.)
('we augment SA with RL-optimisable components, instead of simply using them as standalone algorithms that only interact via shared solutions')
next state을 받기위해서 temprature에 따라 다시 accept와 reject가 결정됨


**contextual combinatorial bandits(CCMAB)**

강화학습은 아니지 action에 따라 변화하는 state가 아닌 환경에서 사용할 수 있다. 어느 조합을 선택해도 다음 state에 아무 영향이 없다.
또한 사용가능한 Arm의 풀이 지속적으로 변하는 volatile arms상황을 고려한다. (action space가 변화하는 것으로 강화학습에서 action masking으로 구현가능하다.)

예를 들어 이런 상황에서 사용가능하다.
해당 state와 관련있는 특징이 여러가지고 이 특징을 엮어 조합을 만들어서 state에 대해 설명해야할때
곰의 이미지가 state이고 특징으로 겨울잠, 꿀(?), 포유류, 연어 etc의 특징이 있으나 정말 관련없는 특징(비행기, 안경)으로 조합을 만들고싶지 않고 이런 search space를 줄이고 싶을때
State마다 사용가능한 arm이 달라지는 것이다.(이는 위의 LeNSE에서 search space를 줄이는 것과 같은 효과를 줄것같다.)


- 미탐색된 arm의 집합(q)이 선택가능한 arm의 개수 B보다 크면 랜덤하게 B개를 고름 (exploration)
- 미탐색된 arm의 집합(q)이 선택가능한 arm의 개수 B보다 작으면 B-q 만큼 고르고 나머지는 탐색된 arm에서 greedy하게 고른다.(exploration+exploitation)
- 미탐색된 arm이 없으면 그냥 B개 선택(exploitation)
- 사용된 예제) 시간에따라, 사용자의 위치에따라 바뀜) 사용가능자-리뷰가능 사업체가 하나의 arm이된다.
- 리워드는 각 리뷰의 품질을 계산한다.(댓클 텍스트 길이와 받은 투표수를 곱하여 계산됨)

그냥 UCB보다 성능이 좋았다고한다.
주어진 정보를 바탕으로 선택을 하고 그 선택이 좋았는지를 알 수 있게된다. exploit-explore를 적절히 수행한다.
