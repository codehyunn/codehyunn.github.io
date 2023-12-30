---
title: "[알고리즘] 서로소 집합 : Union-Find 알고리즘"
header:
  #image: /assets/images/page-header-image.png
  #og_image: /assets/images/page-header-og-image.png
categories:
  - Algorithm
tags:
  - Union-Find
last_modified_at: 2023-12-30
toc: true
toc_sticky : true
---
**🧚🏻‍♀️참고**<br>
이 포스트는 '이것이 취업을 위한 코딩 테스트다 with Python'으로 공부한 내용을 정리한 포스트입니다.
{:.notice--blog-theme}

## | 서로소 집합

서로소 집합은 공통원소가 없는 두 개의 집합을 의미한다. 서로소 집합 자료 구조는 서로소 집합을 바탕으로 한다. 서로소 부분 집합의 데이터를 처리하기 위한 자료 구조가 바로 서로소 집합 자료 구조이다. 서로소 집합 자료구조를 조작하기 위해 **집합을 합치는 union**과 **원소를 찾는 find**, 두 연산이 필요하다.

## | Union-Find 알고리즘
### 알고리즘 표로 정리하기

| 구분 | 내용 |
| :------------------ | :----------------------------- | 
| 설명 | 집합을 **트리 구조**로 표현한다.<br>주어진 노드의 루트 노드를 각각 찾고(find), 이를 비교하여 같은 집합에 속하는지 확인한다. 만약 루트 노드가 다르다면 두 집합을 합친다(union). |
| 알고리즘 | 1. 모든 노드의 부모 노드를 자기 자신으로 초기화한다.<br>2. 부모 노드 테이블을 재귀적으로 거슬러 올라가며 노드의 루트 노드를 찾는다(find). <br>3. 루트 노드가 다르다면 더 큰 루트 노드가 작은 루트 노드를 가리키도록 부모 노드 테이블을 업데이트 한다(union). |
| 시간복잡도 | 코드 구현에 따라 차이가 있을 수 있지만, 아래 코드의 경우 다음과 같다. <br>$$O(V+M(1+log_{2-M/V}V))$$ <br> *이때, V는 노드(Vertex)를 의미하며, M은 최대 find 연산 수를 의미함.  |

### Python으로 구현하기

```python
import sys

sys.stdin = open('answer.txt', 'r')
input = sys.stdin.readline

def find_parent(parents, x) :
  # find 연산 수행하는 함수
  if parents[x] != x :
      parents[x] = find_parent(parents, parents[x])
  return parents[x]
    
def union(parents, x,y) :
  # union 연산 수행하는 함수
  x = find_parent(parents, x)
  y = find_parent(parents, y)
  
  if x > y :
      parents[x] = y
  else :
      parents[y] = x

v, e = map(int, input().split())
parents = [i for i in range(v+1)]

for _ in range(e) :
    a, b = map(int, input().split())
    union(parents,a,b)

print('원소가 속한 집합')
for i in range(1, v+1) : 
    print(find_parent(parents, i), end=' ')

print('\n\n부모 테이블')
for i in range(1, v+1) :
    print(parents[i], end=' ')

###################
# answer.txt
# 6 4
# 1 4
# 2 3
# 2 4
# 5 6

# output
# 원소가 속한 집합
# 1 1 1 1 5 5

# 부모 테이블
# 1 1 1 1 5 5

```