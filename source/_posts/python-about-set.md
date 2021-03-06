---
title: "[python] set에 관한 두가지 사실"
date: 2016-02-28 01:47:45
tags:
- python
- set
---
중복이 제거된 자료구조를 만들어야 할 일이 있어 set을 쓰게 되었는데요.
set으로 만들어진 자료를 바탕으로 루프를 돌며 set안의 item들을 처리하는 작업이었습니다.
item이 너무 많아지게 되면, 메모리 등의 성능 이슈가 발생할 것을 대비하여 20만개씩 잘라서 분절해야 했는데요.
이 때 첫번째 문제가 발생했습니다. python에서 set은 분절 할 수 없다는 것입니다.
리스트의 경우에는 아래와 같이 분절이 가능합니다만, set은 리스트 처럼 분절이 불가합니다.

```
temp_list = ['a, 'b', 'c', 'd', 'e', 'f']
temp_list[:3]      

# ['a, 'b', 'c'] 출력
```

```
temp_set = {'a, 'b', 'c', 'd', 'e', 'f'}
temp_set[:3]

# 아래의 에러 발생
# Traceback (most recent call last):
# File "<stdin>", line 1, in <module>
# TypeError: 'set' object is not subscriptable
```

루프를 돌면서 처리할 수도 있겠지만, 그것보다는 왜 set이 분절이 안될까라는 고민에 빠져 도대체 set은 어떻게 만들어진건가 궁금해지더라구요.
구글링을 하다 파이썬 공식 문서에(2.7 버전, 파이썬 3보다 설명이 괜찮더라구요) set은 아래와 같은 내용을 찾았습니다.

> The sets module provides classes for constructing and manipulating unordered collections of unique elements. Common uses include membership testing, removing duplicates from a sequence, and computing standard math operations on sets such as intersection, union, difference, and symmetric difference. 
> 
> Like other collections, sets support x in set, len(set), and for x in set. Being an unordered collection, sets do not record element position or order of insertion. Accordingly, sets do not support indexing, slicing, or other sequence-like behavior.
> 
> ......
> 
> The set classes are implemented using dictionaries. Accordingly, the requirements for set elements are the same as those for dictionary keys; namely, that the element defines both __eq__() and __hash__(). As a result, sets cannot contain mutable elements such as lists or dictionaries


대략적인 내용을 살펴보면(발번역 및 의역), 


> set은 순서가 없는 중복이 불가능한 collection 자료형이다. 주로 item 테스트, 중복제거 등에 사용되고 교집합, 합집합, 차집합 등을 수학적인 계산이 가능하다. 다른 collection 자료형 처럼 item 검사, 길이, 루프가 가능하다. set은 삽입된 item의 위치를 저장하지 않기 때문에 item 간의 순서가 없다. 따라서 indexing이 불가능하고, 자르기가 안되고, 그외의 sequential한 작업이 불가하다. set은 dictionary를 구현한 클래스인데, dictionary의 key가 set의 item이 된다. 그렇기 때문에 set은 dictionary나 list처럼 중복되는 요소를 담을 수 없다.

정도가 되겠네요. 위의 번역을 토대로 set이 분절이 안되는 이유는 item을 저장할때 순서값을 저장하지 않기 때문이라 할 수 있겠네요. 
순서가 없으니 어디서부터 어디까지라는 것을 정할 수 없을 것이고, 그렇기 때문에 분절이 불가능하다 할 수 있겠습니다.    


그리고 두 번째 문제가 있었는데, set의 루핑 속도에 대한 것이었습니다.
파이썬에서 set역시 list와 같이 iterable한 자료구조이긴 하지만, list에 비해서 iteration 성능은 떨어지는 것으로 익히 알려져 있습니다.
대신 hash 기반으로 만들어지기 때문에 검색 속도는 list에 비해 훨씬 뛰어나죠.

우선 진짜 set이 list에 비해 iteration 성능이 떨어지는지 확인해 보았습니다.

```
from timeit import timeit

def iter(iterable):
    for i in iterable:
        pass

timeit("iter(iterable)", number=1000, setup="from __main__ import iter; iterable=set(range(1000000))")
# 결과 : 14.660961047000455 (단위: 초)

timeit("iter(iterable)", number=1000, setup="from __main__ import iter; iterable=list(range(1000000))")
# 결과 : 11.929272602999845 (단위: 초)
```

1,000,000(백만, 0~999,999) 까지의 숫자를 담은 set을 1,000번 반복한 결과 14.66초 정도가 걸렸습니다.
반면에 같은 조건으로 list를 1,000번 반복한 결과 11.93초 정도가 걸렸네요.
대략 3초 정도가 차이납니다.
사실 1,000,000을 1,000번 반복하여 총 1,000,000,000(10억)번 루프를 돌아야만 3초 정도(14-11)가 차이가 난것이지,
현업에서 사용하는 정도의 크기로 비교해보면 그렇게 큰 차이가 나지는 않는것 같네요.
속도차이가 크게 나지 않기 때문에 특별한 이유가 없다면 set으로 루프를 돌려도 무방할 것 같습니다.

글을 쓰고 나니 알맹이는 없고 횡설수설한 느낌입니다.
어찌되었든 위의 두가지(set은 slice를 못하는 것, list보다 iteration 성능이 떨어지는것) 사실로 set에 대한 궁금증이 완전히 사라진 것은 아닙니다. 
set이라는 자료구조가 머리속에 그림으로 그려져서 더하고 삭제하거나 탐색할때, 또는 set을 list로 바꿀 때와 같은 경우를 한번에 떠올릴 수 있으면 참 좋겠습니다만, 아직까지는 더 노력이 필요한 것 같습니다. 
너무 많은 내용을 한번에 다루면 포스팅하는 저나 읽는 사람들 모두 피곤할테니 일단은 여기서 글을 마무리 하는게 좋을것 같네요.
