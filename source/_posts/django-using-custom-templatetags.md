---
title: "[django] 커스텀 템플릿태그(templatetags) 활용하기"
date: 2016-03-03 00:34:12
tags:
- django
- python
- templatetags
---
웹 개발을 하다보면, html 코드 상에서 다양한 연산을 해야하는 경우가 발생합니다.
그래서 php, jsp, asp, jade 등 각 언어별 웹 프레임워크에서 이와 같은 경우를 처리해주기 위한 기능을 제공하고 있습니다.
장고(django) 템플릿(template)에서도 위와 같은 웹 프레임워크와 같이 동일한 기능을 지원하는 템플릿태그(templatetags)라는 것이 있습니다.
장고의 템플릿태그는 다른 웹 프레임워크와 마찬가지로 기본적으로 개발자가 필요한 기능은 대부분 제공하고 있습니다.

웹 프레임워크가 기본적인 기능을 대부분 제공하고 있지만, 개발을 하다보면 자신이 원하는 기능이 없는 경우가 간혹 있습니다.
그래서 장고에서는 개발자가 커스텀으로 템플릿태그를 만들수 있는 기능을 제공하고 있습니다.

우선 아래와 같이 앱(app) 아래에 templatetags라는 폴더를 만들어 줍니다. 
temaplatetags라는 폴더 이름은 고정값이므로 반드시 동일하게 생성합니다.

```
proj/
	app/
		__init__.py
		models.py
		view.py
		templatetags/
			__init__.py
			custom_tags.py    (커스텀 템플릿태그를 저장할 모듈 파일)
```

이때,  app은 반드시 setting 파일의 INSTALLED_APPS에 추가가 되어 있어야 합니다.
그리고 한가지 주의할 점은 여러 앱에 각각 templatetags가 있는 경우, 모듈의 이름이 겹치지 않도록해야 합니다.
이유는, template에서 커스텀 태그는 앱의 위치와 상관없이 모듈 이름으로 로드되므로 이름이 겹치게 되면 충돌이 발생하게 됩니다.
즉,

```
proj/
	app/
		templatetags/
			custom_tags.py
	common/
		templatetags/
			common_tags.py    (태그모듈 이름이 겹치지 않도록 주의!!!)
```

태그 모듈을 사용하는 방법은 간단합니다.
커스텀 태그를 사용하고자하는 템플릿 파일의 상단에 아래와 같이 한줄만 추가해주면 됩니다.

```
{% load custom_tags %}
```

그렇다면 실제로 커스텀 태그를 만들어서 사용하는 예제를 만들어 보도록 하겠습니다.
custom_tags.py 파일을 열어 사용하려는 태그 이름으로 메서드 이름으로 지정하여 만들어줍니다.

```
@register.filter            # 1
def add_str(left, right):
return left + right

@register.simple_tag            # 2
def today():
return datetime.now().strftime("%Y-%m-%d %H:%M")

@register.assignment_tag            # 3
def max_int(a, b):
return max(int(a), int(b))

@register.inclusion_tag(div.html, takes_context=True)            # 4
def include_div(context): 
return {
'div_param': context['param']
}
```

대략 위의 4가지 태그로 구분할 수 있는데, 각 태그에 따라서 사용법이 조금씩 다릅니다.
이 4가지 커스텀 태그만 이용하면 웬만한 기능은 다 만들어 낼 수 있습니다.
번호별 사용법은 아래와 같습니다.

```
# 1.
# filter 태그는 앞의 값(left)에다가 뒤의 값(right)을 연산하는 태그입니다. 
# filter이기 때문에 여러개의 필터를 붙여서 사용가능합니다.
# add_str 메서드의 left 파라미터가 prefix에 해당하고, right 파라미터가 url에 해당합니다. 
# 결과적으로 prefix + url이 add_str 메서드를 통해 div의 text가 되는 것이지요.

<div>
{{ prefix | add_str: url | add_str: name | add_str: params }}
</div>

# 2. 
# simple_tag는 단순히 어떤 특정값을 출력합니다.
# 아래와 같이 today를 입력하면, "2016-3-2 10:00"과 같이 현재 시간이 출력됩니다.

<div>
{{ today }}
</div>

# 3. 
# assignment_tag는 템플릿에서 사용가능한 변수에 결과를 저장하는 역할을 합니다. 
# 어찌보면 with 태그와 유사한 형태라 할 수 있으나, with과는 다르게 {% endwith %} 처럼 끝을 맺어줄 필요가 없습니다.
# 즉, 좀 더 간편하게 변수를 설정해 줄 수 있고, 필요한 기능을 태그 모듈에 별도로 삽입할 수 있다는 장점이 있습니다.

{% max_int first_count second_count as max_count %} 

# 4.
# inclusion_tag는 저도 프로젝트에 직접 사용해 보진 않았지만, 테스트는 해보았습니다.
# 간략히 설명해서 inclusion_tag를 사용하면 데코레이터의 첫번째 파라미터인 템플릿을 호출하여 부모 템플릿에 출력합니다.
# 이때, 호출되는 템플릿에 부모 템플릿(호출하는 템플릿)의 각종 파라미터를 전달해 줄 수 있습니다.
# 데코레이터의 takes_context=True로 설정해주면,
# 부모 템플릿의 context의 값을 가져와 호출하는 템플릿으로 전달할 수 있습니다.

parent.html
{{ include_div}}

div.html
<div>{{ div_param }}</div>
```

이상 커스텀 태그의 종류와 사용법에 대해서 알아보았습니다. 
제가 설명드린 커스텀 태그는 아주 기초적인 부분이라 제작 및 사용법이 아주 간단한데요.
커스텀 태그 파일은 파이썬 모듈이기 때문에 파이썬에서 사용할 수 있는 내장함수와 모든 확장 모듈을 사용할 수 있기 때문에,
얼마든지 복잡하고 파워풀한 기능을 가진 태그를 만들어 낼 수 있습니다.

하지만, 복잡한 연산을 처리하는 것은 템플릿보다는 웹서버 단에서 처리하는 것이 우선이고,
서버에서 처리가 곤란하거나 불가피한 상황인 경우에 태그를 사용해서 처리하는 것이라 생각합니다.
아마 장고에서도 사용가능한 기본 태그를 최소한으로 만들어 놓은 것도 같은 이유 때문일 거라 생각이 드네요.
