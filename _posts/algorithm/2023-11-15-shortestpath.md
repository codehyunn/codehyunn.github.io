---
title: "[알고리즘] 최단 경로 알고리즘"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Algorithm
tags:
  - Shortest Path
last_modified_at: 2023-11-16
toc: true
toc_sticky : true
---
**🧚🏻‍♀️참고**<br>
이 포스트는 '이것이 취업을 위한 코딩 테스트다 with Python'으로 공부한 내용을 정리한 포스트입니다.
{:.notice--blog-theme}

## | 최단 경로 알고리즘
최단 경로 알고리즘은 그래프 상에서 노드 간의 탐색 비용을 최소화하는 알고리즘으로, **다익스트라(Dijkstra), 플로이드 워셜(Floyd Warshall), 벨만 포드(Bellman Ford) 알고리즘**이 여기에 속한다. 교재에는 다익스트라 알고리즘과 플로이드-워셜 알고리즘만 언급됐지만, 백준을 풀다보니까 벨만-포드 알고리즘이 필요한 경우도 종종 있어서 같이 정리해봤다.

## | 다익스트라 알고리즘
### 알고리즘 표로 정리하기

| 구분 | 내용 |
| :------------------ | :----------------------------- | 
| 설명 | 특정한 **하나**의 노드에서 **다른 모든** 노드까지의 최단 거리를 구하는 알고리즘<br>항상 현재 노드에서 가장 가까운 노드를 선택하여 과정을 진행하기 때문에 **그리디 알고리즘**에 포함되며,<br>한 단계마다 하나의 노드에 대한 최단 거리를 확실히 구한다는 특징이 있다. <br>가중치가 **음수이면 안된다는 조건**을 가진다.|
| 알고리즘 | 1. 시작 노드와 연결된 노드는 간선의 가중치로, 나머지는 큰 수로 최단 거리 테이블을 초기화한다.<br>2. 방문하지 않은 노드 중에서 최단 거리를 가지는 노드를 현재 노드로 선택한다. (그리디)<br>3. 현재 노드를 거쳐 다음 노드로 가는 거리와 테이블에 저장된 거리를 비교한 뒤, 작은 값으로 테이블을 업데이트한다.<br>4. 모든 노드를 방문하여 업데이트할 게 없을 때까지 2-3을 반복 수행한다. |
| 구현 | **우선순위큐(heapq)**를 사용하여 구현한다. |
| 시간복잡도 | $$O(log(ElogV))$$ <br> *이때, E는 간선(Edge)를 의미하며, V는 노드(Vertex)를 의미함.  |

### Python으로 구현하기

```python
import sys
import heapq

sys.stdin = open('answer.txt', 'r')
input = sys.stdin.readline

INF = int(1e9)

n, m = map(int, input().split()) # n, m = 노드, 간선의 개수
start = int(input()) # start = 시작 노드

# 그래프 정보 입력 받아 (연결된 노드, 거리) 순으로 저장
graph = [[] for _ in range(n+1)] 
for _ in range(m) :
    a,b,dist = map(int,input().split())
    graph[a].append((b,dist))
    
# 최단거리 테이블(현재까지의 최단 거리를 저장) 초기화
distance = [INF for _ in range(n+1)] 
distance[start] = 0

# 노드들을 거리가 가까운 순으로 정렬하기 위해 (거리, 노드) 순으로 저장
queue = [(0, start)] 

# 우선순위 큐를 활용한 다익스트라 알고리즘
while queue :
    curr_dist, curr_node = heapq.heappop(queue)
    if distance[curr_node] < curr_dist :
        # heapq는 항상 가장 작은 값(거리)을 pop함
        # 그러므로 방문한 노드의 경우, 테이블에 저장된 값이 heapq에서 pop한 값보다 작음
        continue

    for next_node, next_dist in graph[curr_node] :
        if distance[next_node] > curr_dist + next_dist : 
            # 테이블에 저장된 거리(distance[next_node])와 
            # 현재 노드를 거쳐 다음 노드로 가는 거리(new_dist) 비교, 업데이트         
            distance[next_node] = curr_dist + next_dist
            heapq.heappush(queue, (curr_dist + next_dist, next_node))

# 최단거리 테이블 출력 (1번부터 n번까지)    
print(distance[1:])

###################
# answer.txt
# 6 11
# 1
# 1 2 2
# 1 3 5
# 1 4 1
# 2 3 3
# 2 4 2
# 3 2 3
# 3 6 5
# 4 3 3
# 4 5 1
# 5 3 1
# 5 6 2

# output
# [0, 2, 3, 1, 2, 4]
```

## | 플로이드 워셜 알고리즘
### 알고리즘 표로 정리하기

| 구분 | 내용 |
| :------------------ | :----------------------------- | 
| 설명 | **모든 노드 간**의 최단 경로를 구하는 알고리즘<br>현재 노드를 거치는 경우의 수를 모두 고려하며, **다이나믹 프로그래밍**에 포함된다.<br>점화식으로 표현하면 다음과 같다. $$D_{ab} = min(D_{ab}, D_{ak}+D_{kb}) $$ |
| 알고리즘 | 1. 최단 거리 테이블을 초기화한다.<br>2. 모든 노드에 대하여, 현재 노드를 거쳐 다음 노드로 가는 거리와 테이블에 저장된 거리를 비교하여 작은 값으로 테이블을 업데이트한다. <br>3. 모든 노드가 2.의 현재 노드가 될 수 있도록 노드 수만큼 2.를 반복한다.
| 구현 | **삼중 For문**으로 구현한다. |
| 시간복잡도 | $$O(V^3)$$ <br> *이때, E는 간선(Edge)를 의미하며, V는 노드(Vertex)를 의미함.  |

### Python으로 구현하기
```python
import sys

sys.stdin = open('answer.txt', 'r')
input = sys.stdin.readline

INF = int(1e9)

n, m = map(int, input().split()) # n, m = 노드, 간선의 개수

# 그래프 정보 입력 받아 거리 업데이트 (graph[a][b] = 노드 a에서 b까지의 거리)
distance = [[INF for _ in range(n+1)] for _ in range(n+1)] 
for _ in range(m) :
    a,b,dist = map(int,input().split())
    distance[a][b] = dist

# 같은 노드로 가는 경우의 거리를 0으로 초기화
for i in range(1, n+1) :
    distance[i][i] = 0

# 나머지 경우에 대해 최단 거리 구하기
for k in range(1, n+1) :
    for i in range(1, n+1) :
        for j in range(1, n+1) :
            distance[i][j] = min(distance[i][k] + distance[k][j], distance[i][j])
    
# 모든 경로에 대한 최단 경로 출력
for d in distance[1:] :
    print(d[1:])

###################
# answer.txt
# 4 7
# 1 2 4
# 1 4 6
# 2 1 3
# 2 3 7
# 3 1 5
# 3 4 4
# 4 3 2

# output
# [0, 4, 8, 6]
# [3, 0, 7, 9]
# [5, 9, 0, 4]
# [7, 11, 2, 0]
```

## | 벨만 포드 알고리즘
### 알고리즘 표로 정리하기

| 구분 | 내용 |
| :------------------ | :----------------------------- | 
| 설명 | 특정한 **하나**의 노드에서 **다른 노드**까지의 최단 거리를 구하는 알고리즘<br>매 단계마다 모든 간선을 확인한다는 특징이 있으며 **다이나믹 프로그래밍**에 포함된다.<br>다익스트라 알고리즘과 달리 가중치가 **음수인 경우에도 사용 가능**하지만, 다익스트라 알고리즘보다 시간복잡도가 높은 편이다.|
| 알고리즘 | 1. 간선을 하나 선택한다.<br>2. 선택한 간선을 거쳐 다음 노드로 가는 거리와 다음 노드 테이블에 저장된 거리를 비교하여 테이블을 업데이트 한다.<br>3. 모든 간선에 대해 1-2를 반복한다.<br>4. 노드의 수만큼 3.을 반복하여 최단 거리를 찾고, 음의 사이클 존재 여부를 파악한다. |
| 구현 | **2중 For 문**을 사용하여 구현한다. |
| 시간복잡도 | $$O(EV)$$ <br> *이때, E는 간선(Edge)를 의미하며, V는 노드(Vertex)를 의미함.  |

V개의 노드가 존재한다고 할 때, 모든 간선 확인(위 표의 '알고리즘' 3.)을 V-1번을 반복하면 최단 경로 업데이트가 완료된다.<br>다시 말해 V번째 반복에서도 업데이트가 이뤄진다면 그래프 상에 음의 사이클이 존재한다고 할 수 있다.<br>그러므로 모든 간선 확인을 V번 반복하면 최단 경로 갱신을 마치고 음의 사이클 존재 여부도 확인 가능하다. ꒰⍤꒱
{:.notice}

### Python으로 구현하기
```python
import sys

sys.stdin = open('answer.txt', 'r')
input = sys.stdin.readline

INF = int(1e9)

n, m = map(int, input().split()) # n, m = 노드, 간선의 개수
start = int(input()) # 시작 노드

# 그래프 정보 입력 받기
graph = [] # 간선을 선택하므로 일차원 리스트에 입력받는다.
for _ in range(m) :
    a,b,dist = map(int,input().split())
    graph.append((a,b,dist))

# 최단 거리 테이블 초기화
distance = [INF for _ in range(n+1)]
distance[start] = 0

# 최단 거리 구하기
for vertex in range(1,n+1) :
    for edge in graph :
        curr_node, next_node, dist = edge
        if distance[curr_node] != INF and distance[next_node] > distance[curr_node] + dist:
            distance[next_node] = distance[curr_node] + dist
            
            if vertex == n :
                # n번째 반복에서 업데이트가 이루어지면 음의 사이클이 존재함
                print('음의 사이클 존재')
                exit()
    
# 최단 경로 출력(음의 사이클이 존재하지 않는 경우)
print(distance[1:])

###################
# answer.txt (1)
# 3 4
# 1
# 1 2 4
# 1 3 3
# 2 3 -4
# 3 1 -2

# output
# 음의 사이클 존재

###################
# answer.txt (2)
# 3 4
# 1
# 1 2 4
# 1 3 3
# 2 3 -1
# 3 1 -2

# output
# [0, 4, 3]
```