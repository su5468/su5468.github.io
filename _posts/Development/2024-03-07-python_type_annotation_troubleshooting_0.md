---
layout: posts
categories:
    - Development
title: 파이썬 타입 어노테이션 트러블슈팅 0
---

# Intro

파이썬은 대표적인 동적 타이핑 언어이다.  
그 말은, 변수의 타입을 마음대로 바꿔도 그 자체로는 문제가 되지 않는다는 뜻이다.  
하지만, 유지보수 등을 위해서는 타입이 정적이고 명시적인 게 좋을 수 있다.  
이때, 파이썬의 변수나 매개변수, 함수 반환값에 대해 타입을 명시하는 것이 타입 어노테이션이다.  
물론 파이썬이 동적 타이핑 언어라는 철학은 변하지 않으므로 어노테이션 자체는 아무런 역할을 하지 않는다.  
즉, 타입 어노테이션은 어디까지나 서드파티 타입 체커를 위해 제공되는 기능이다.  

# typing.TYPE_CHECKING
기타 기본적인 내용은 구글링해도 잘 나올 것이라 믿고,  
여기서 다루고 싶은 것은 런타임에 불필요하게 모듈을 불러오지 않고 타이핑을 하고 싶을 때를 위한 트러블슈팅이다.  

이를테면 내가 선언하고 싶은 어떤 데이터의 타입(클래스)이 정의된 모듈을 직접 불러오고 싶지 않을 수 있다.  
물론 어떻게든 그 클래스의 인스턴스를 만들기 위해서는 어차피 해당 모듈을 명시적으로 임포트하지 않더라도 임포트되겠지만,   
가독성 문제라든지, 모듈을 불러오는 데에 있어서 순환 참조 문제라든지... 를 해결하기 위해서는 실제 런타임에 필요한 구문만 남기고 싶을 수 있을 것이다.  

이때는 `typing` 모듈에서 `TYPE_CHECKING` 상수를 가져오면 된다.  
`TYPE_CHECKING`은 별도의 타입 체커를 실행할 때만 `True`이고, 런타임에서는 `False`인 상수이다.  
즉, 아래와 같은 형태로 구성하면 타입 체킹에 필요한 클래스를 지닌 모듈들을 구분할 수 있다.  

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
  from some_module import SomeClass
```

이렇게 한다면, 실제 런타임에서는 불필요하거나 순환 참조를 일으킬 수 있는 모듈을 불러오지 않을 수 있다.  

다만, 이 경우 런타임에선 `SomeClass`가 임포트되지 않았다는 점에 주의해야 한다.  
아래와 같은 구문은 오류를 일으킬 것이다.  

```python
spam: SomeClass
```

런타임에선 `SomeClass`를 불러오지 않고, 따라서 어노테이션을 해석하는 과정에서 오류가 날 것이다.  
따라서 전방 참조(Forward Reference)를 사용해야 한다.  

# 전방 참조

전방 참조란, 아직 없는 값을 미리 참조한다는 뜻이다.  
불러오지 않은 클래스를 어노테이션에 사용하기 때문에, 전방 참조가 필요하다.  

`typing` 모듈에서 가져온 타입 앨리어스(`typing.List` 같은)에서 잔방 참조를 하는 경우 그냥 문자열로 감싸주면 된다.  

```python
from typing import List

egg: List[SomeClass]    # Before
egg: List["SomeClass"]  # After
```

그런데 이 글을 작성하면서 알았는데, 이러한 기본 타입들의 앨리어스는 전부 Deprecated 될 예정이라고 한다.  
즉, 이제는 그냥 `list[SomeClass]` 가 맞는 어노테이션인데, 공식문서에서 이럴 경우 전방 참조에 문자열을 사용하는 것을 틀린 것으로 보고 있다.  
정확히는, 아래와 같이 해야 한다.  

```python
from typing import ForwardRef

egg: list[SomeClass]                # Before
egg: list[ForwardRef("SomeClass")]  # After
```

생각해보면, 딕셔너리 같은 타입은 `dict["SomeClass"]`가 타입 어노테이션인지, `"SomeClass"`를 키로 갖는 값을 찾는 것인지 모호하므로 당연한 일일 것이다.  
아무튼, 요약하자면 아래와 같다.  

1. `typing.List`와 같은 `typing` 모듈 앨리어스를 사용하면 문자열로 타입을 참조하기.
2. `list`와 같은 실제 클래스를 사용하면 `typing.ForwardRef()`를 사용해서 타입을 참조하기.

# from __future__ import annotations

그런데, 이 방법도 조금 복잡하다.  
전방참조니 뭐니 헷갈리는 경우도 많고, 일관적이지도 않아서 가독성도 떨어질 것이다.  

가장 간단한 방법은, `from __future__ import annotations`를 사용하는 것이다.  
이 경우, 타입 어노테이션은 런타임에 해석되지 않는다.  
정확히는, 그 값이 해석되는 대신 `__annotations__`에 문자열 형태로 저장된다.  
`__future__`에 존재하는 기능은 추후 파이썬에서 공식적으로 받아들여질 기능이므로, 사용하는 것이 권장된다 할 수 있다.  
*(물론 하위호환성을 잘 생각할 필요는 있겠지만...)*  

아무튼, `annotations`를 임포트하는 경우 그냥 전방 참조는 무시하고 아래와 같이 하면 된다.  

```python
from __future__ import annotations

egg: list[SomeClass] # 타입 어노테이션 해석되지 않으므로 에러 X.
```

# 결론

타입 어노테이션을 위한 불필요한 모듈을 런타임에는 로드하지 않고, 간편하게 사용하는 방법은 아래와 같다.  

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
  from some_module import SomeClass

spam: SomeClass
egg: list[SomeClass]
```

# 참조

[파이썬 공식문서 - typing](https://docs.python.org/ko/dev/library/typing.html)  
[PEP 563](https://peps.python.org/pep-0563/)  
[PEP 585](https://peps.python.org/pep-0585/)  
