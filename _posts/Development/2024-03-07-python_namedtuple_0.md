---
layout: posts
categories:
    - Development
title: 파이썬 네임드튜플(NamedTuple) 0
---

# Intro

살다 보면 튜플의 각 원소에 이름이 없어서 불편할 때가 있다.  
예를 들어, Pillow나 PyAutoGui같은 라이브러리는 이미지의 좌표나 크기 정보를 인자로 넘겨줄 일이 종종 있다.  
이때, 튜플을 사용하게 되는데, 이러한 값(좌표의 x축과 y축, 크기의 너비와 높이 등)들을 튜플로 묶어서 관리하는 건 좋지만, 
가끔은 튜플 내부의 값에 인덱싱으로만 접근할 수 있다 보니 불편하고 비직관적이라는 생각이 든다.  
네임드튜플은 이름처럼 튜플의 각 원소에 이름을 정해줄 수 있다.  

# collections.namedtuple()

우선, `collections` 모듈의 `namedtuple()`을 이용해서 네임드튜플을 만들 수 있다.  
네임드튜플 자체는 (당연히) 일종의 클래스이고, `namedtuple()`은 정확히 말하자면 네임드튜플의 서브클래스를 만드는 함수이다.  
풀어서 설명하자면, `namedtuple()`은 인스턴스를 만드려고 쓰는 게 **아니라**, 새로운 클래스를 만들기 위해 사용한다는 것이다.  

```python
from collections import namedtuple

# 네임드튜플 클래스를 만드는 구체적인 설명은 공식문서 참조.
Position = namedtuple("Position", ["x", "y"])  # 새로운 네임드튜플 클래스.

# Position의 인스턴스 생성.
pos1 = Position(0, 0)
pos2 = Position(24, 1)
```

네임드튜플은 겉보기엔 일반적인 클래스랑 크게 다르지 않아 보인다.  
하지만, 이 친구들은 튜플의 서브클래스기 때문에 인덱싱 등을 이용해서 튜플처럼 사용이 가능하고, 또한 불변(Immutable)이다.  
앞에서 이야기했듯, 살다 보면 튜플을 클래스처럼 쓰고 싶을 때가 있는데, 그때 아주 간편하게 쓸 수 있는 기능 되시겠다.  

```python
# 위 코드에 이어서

pos = Position(12, 4)
pos.x                  # 12
pos.y                  # 4
pos[0]                 # 12
pos[1]                 # 4
```

# typing.NamedTuple()

네임드튜플의 타입 힌팅은 어떻게 가능할까?  
`typing` 모듈엔 타입 어노테이션을 지원하는 업그레이드된 네임드튜플이 있다.  

```python
from typing import NamedTuple

class Position(NamedTuple):
  x: int
  y: int

pos = Position(6, 21)
```

일반 클래스 문법과 많이 비슷해진 것을 볼 수 있다. 데이터클래스(Dataclass)가 생각나기도 하고...  
튜플에 기반하는 간단하고 편리한 클래스를 빠르게 만들 수 있다는 점에서 데이터클래스와는 차이가 있다.  
서문의 예시를 기억해보자, 튜플을 인자로 받는 함수에 데이터클래스나 일반 클래스를 넘겨주려면 해야 할 일이 많을 것이다...

# 결론

네임드튜플은 튜플을 사용해야 하는 상황에서 간편하고 읽기 쉬운 코드를 작성하는 데에 사용할 수 있다.  
타입 힌팅이 필요할 때 기준 아래와 같이 사용하면 된다.  

```python
from typing import NamedTuple

class SomeClass(NamedTuple):
  spam: int
  egg: str

some_var = SomeClass(1, "eggtart")
some_var.spam
some_var[0]
```
