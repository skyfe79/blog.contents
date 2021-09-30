---
title: "파이썬 데코레이터 작성시 주의할 점"
date: 2021-09-30T09:19:38Z
draft: false
tags: ["python"]
---

`출처`: [효율적 개발로 이끄는 파이썬 실천 기술](http://www.yes24.com/Product/Goods/99123748)

파이썬 데코레이터는 주로 아래와 같은 패턴으로 작성한다.

```python
def deco(f):
    def wrapper(*args, **kwargs):
        v = f(*args, **kwargs)
        return v
    return wrapper
```

만약 데코레이터가 인자를 받는다면 데코레이터를 반환하는 함수를 작성한다.

```python
def deco(z):
    def _deco(f):
        def wrapper(*args, **kwargs):
            v = f(*args, **kwargs)
            return v
        return wrapper
    return _deco
```

이 데코레이터를 사용해 보자.

```python
@deco
def my_func():
    print('Hi!')

>>> my_func.__name__
wrapper
```

my_func의 실제 이름이 데코레이터에 의해서 wrapper가 된다. 여러 위치에서 동일한 데코레이터를 사용하면 같은 함수명으로 여러 처리가 존재하므로 버그가 발생했을 때 원인 조사가 어려워 진다. 

이럴 때, functools.wraps() 데코레이터를 사용하면 좋다. `@wraps(f)`는 실제로 실행될 함수의 이름으로 치환한다.

```python
from functools import wraps

def deco(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        v = f(*args, **kwargs)
        return v
    return wrapper

@deco
def func():
    """func입니다"""
    print('exec')

>>> func.__name__
func
```

위 결과에 보이듯이 func.__name__이 wrapper가 아닌 func가 되었다. 데코레이터를 정의할 때는 꼭 functolls.wraps() 를 사용하자.