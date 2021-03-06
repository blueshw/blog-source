---
title: "[번역] 프레젠테이션 컴포넌트와 컨테이너 컴포넌트"
date: 2017-06-26 20:04:59
tags:
- react
- component
- container
---
> 원본 : [https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)  
> 2년이나 지난 글이지만, 컴포넌트를 어떻게 구현해야 하는 문제에 있어서는 이만한 가이드가 없다고 생각해서 번역해보았습니다.  
> 자연스럽지 못한 부분, 의역이 다수 포함되어 있을 수 있습니다.

리액트 어플레케이션을 만들때 제가 찾은 아주 유용하면서 간단한 패턴이 있습니다. 만약 여러분이 [현재 리액트를 사용한다면](https://facebook.github.io/react/blog/2015/03/19/building-the-facebook-news-feed-with-relay.html), 이미 알고 있을지도 모릅니다. [이 페이지](https://medium.com/@learnreact/container-components-c0e67432e005))가 잘 설명해 줄것입니다. 하지만, 저는 몇가지 더 얘기하고 싶네요.

여러분은 컴포넌트를 더 쉽게 재사용할 수 있는 방법과 왜 컴포넌트를 두개의 카테고리로 나눠야 하는지에 대한 이유를 알게 될 것입니다. 이미 들어보았던, Fat and Skinny, Smart and Dumb, Stageful and Pure, 화면과 컴포넌트 등과 같은 개념들이 이미 있지만 저는 이것을 컨테이너와 프레젠테이션 컴포넌트(\*)라 부르겠습니다. 이것들이 모두 동일한 개념은 아니지만, 기본적인 아이디어는 비슷합니다.

### 프레젠테이션 컴포넌트
- 어떻게 보여지는지와 관련있다.
- 프레젠테이션 컴포넌트와 컨테이너 컴포넌트가 모두 그 안에 들어가 있을것(\*\*)이고, 일부 DOM 마크업과 스타일도 가지고 있다.
- 종종 this.props.children을 통해서 노출된다.
- Flux 액션이나 stores 등과 같은 앱의 나머지 부분들에 의존적이지 않다.
- 데이터를 가져오거나 변경하는 방법에 대해서 관여할 필요가 없다.
- props를 통해 배타적으로 callback 함수와 데이터를 받는다.
- 상태를 거의 가지고 있지 않다(만약 상태를 가지고 있다면, 데이터에 관한 것이 아닌 UI 상태에 관한 것이다).
- 만약 상태, 생명주기, hooks, 또는 퍼포먼스 최적화가 필요없다면, [유틸함수](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components)로서 쓰여질것이다.
- 예를들면 페이지, 사이드바, 스토리, 유저정보, 리스트 등이 있다.

### 컨테이너 컴포넌트
- 어떻게 동작하는지와 관련있다.
- 프레젠테이션 컴포넌트와 마찬가지로 프레젠테이션 컴포넌트와 컨테이너 컴포넌트 모두 가지고 있지만 감싼 divs를 제외하고는 DOM 마크업을 가지고 있지 않는다. 스타일 역시 가지고 있지 않는다.
- 데이터와 기능(행동)을 프레젠테이션 컴포넌트와 다른 컴포넌트에 제공한다.
- Flux(or Redux) 액션을 호출하고, 프레젠테이션 컴포넌트에 콜백함수로써 제공한다.
- 데이터 소스 역할을 하기 때문에 상태가 자주 변경된다.
- 직접 만드는것 보단 대게 React Redux의 connect() 함수, Relay의 createContainer() 함수, Flux Utils의 Container.create()와 같은 [Higher Order Components](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)를 이용해서 만들어진다.
- 예를들면 유저페이지, 팔로워 사이드바, 스토리 컨테이너, 팔로우한 유저 리스트 등이 있다. 

저는 이것들을 확실하게 구분하기 위하여 서로 다른 폴더에 생성합니다.

### 장점
- 이 방법으로 컴포넌트를 작성하면 당신의 앱(기능)과 UI에 대한 구분을 이해하기가 더 수월하다.
- 재사용성이 더 뛰어나다. 완전히 서로 다른 상태값과 함께 같은 프레젠테이션 컴포넌트를 사용할 수 있고, 재사용 될 수 있는 별도의 컨테이너 컴포넌트로 변경할 수 있다.
- 프레젠테이션 컴포넌트는 말하자면 앱의 팔레트와 같다. 앱의 싱글페이지 위에서 앱의 로직을 건드리지 않고 디자이너에게 모든 변화를 조정하게 할 수 있다.
- 이것은 사이드바, 페이징, 컨텍스트메뉴와 같은 레이아웃 컴포넌트를 추출하도록 할것이고, 이것은 동일한 마크업이나 몇몇의 컨테이너 레이아웃을 반복해서 작성하는 대신 this.props.children을 통해서 구현될 수 있다. 

> 컴포넌트는 DOM을 생성하지 말아야 합니다. 컴포넌트는 단지 UI와 관련된 것들을 조합하는 것을 제공하는 것이 필요합니다. 

이러한 이점을 당신의 앱에 적용해보세요.

### 언제 컨테이너를 도입해야하나요?
우선 앱을 만들때 프레젠테이션 컴포넌트를 먼저 만드세요. 그러면 너무 많은 props를 중간 컴포넌트로 보내야 한다는 것을 깨닫게 될것입니다. 전달받은 props를 사용하지 않고 아래로 전달하기만 하는 컴포넌트나 자식 컴포넌트가 더 많은 데이터를 필요로 할때 모든 중간 컴포넌트를 재구성해야하는 컴포넌트들이 있다는 것을 알게 될것입니다. 바로 이 때 컨테이너 컴포넌트를 도입해야합니다. 데이터나 아무 상관없는 중간 컴포넌트에 대해 걱정이 없는 leaf 컴포넌트의 행위가 담긴 props를 얻을 수 있는 방법이 될 것입니다.

리팩토링이 진행중이기 때문에 처음부터 도입하려고 시도해서는 안됩니다. 이 패턴을 실험해보려면, 어떤때에 함수를 추출할지를 아는것 처럼 어떤때에 컨테이너를 추출해야하는지를 직감으로 알아야 합니다. 저의 [free Redux Egghead series](https://egghead.io/series/getting-started-with-redux)가 당신에게 도움이 될것입니다.

### 다른 분리방법들
프레젠테이션 컴포넌트와 컨테이너 컴포넌트의 차이는 기술적인 부분이 아니라는 것을 이해하는 것은 중요합니다. 이것은 오히려 용도에 따른 차이입니다.

대조적으로, 여기 기술적으로 관련된 구분이 몇가지 있습니다.

- Stateful and Stateless
어떤 컴포넌트들은 React의 setState() 메소드를 사용한다. 컨테이너 컴포넌트는 상태가 자주 변하는 경향(stateful)이 있고 프레젠테이션 컴포넌트는 그렇지 않은 경향(stateless)이 있다. 다만 이것은 엄격한 규칙은 아니다. 프레젠테이션 컴포넌트에서 상태가 자주 바뀔수도 있고, 컨테이너 컴포넌트에서 상태변화가 없을수도 있다.
- Classes and Function 
[리액트 0.14부터](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components) 컴포넌트를 클래스와 함수 모두로 선언이 가능하다. 함수 컴포넌트는 정의가 간단하지만 클래스 컴포넌트에 비해서 몇가지 부족한 점이 있다. 이러한 제한의 일부는 미래에는 없어질 수도 있지만 현재는 존재한다. 왜냐하면 함수 컴포넌트는 이해하기 쉽기 때문이다. 만약 state, 라이프사이클 후킹 또는 퍼포먼스 최적화가 필요하다면 반드시 클래스 컴포넌트를 사용해야한다. 왜냐하면 이들은 클래스 컴포넌트에서만 사용할 수 있기 때문이다.
- Pure and Impure
만약 같은 props와 state가 주어졌을때 같은 결과가 돌아오는것이 보장된다면 사람들은 컴포넌트가 pure하다고 말한다. 퓨어 컴포넌트는 클래스나 함수로 모두 정의 될수 있습니다. 그리고 stateful하거나 stateless 할수도 있다. 퓨어 컴포넌트의 또다른 중요한 점은 props와 state의 변화에 깊게 관여하지 않다. 그래서 퓨어 컴포넌트의 렌더링 퍼포먼스는 shouldComponentUpdate() 함수의 얕은 비교에 의해 최적화 될 수 있다. 현재는 클래스에서만 shouldComponentUpdate() 함수를 사용할 수 있지만 아마도 나중에는 함수에서는 사용할 수 있을것이다.

프레젠테이션 컴포넌트와 컨테이너 컴포넌트 둘다 어느쪽 컴포넌트에나 들어갈 수 있습니다. 제 경험에 의하면 프레젠테이션 컴포넌트는 stateless한 pure 함수가 되는 경향이 있고, 컨테이너 컴포넌트는 stateful한 pure 클래스가 되려는 경향이 있습니다. 하지만 규칙은 아니고 주목할만한 것입니다. 왜냐하면 저는 구체적인 상황들에서 정확히 반대의 경우로 만들어지는 것을 보았기 때문입니다. 

### 예제
[This Gist](https://gist.github.com/chantastic/fc9e3853464dffdb1e3c) by [Michael Chan](https://twitter.com/chantastic)

### 읽을거리
- [Getting Started with Redux](https://egghead.io/series/getting-started-with-redux)
- [Mixins are Dead, Long Live Composition](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)
- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005)
- [Atomic Web Design](http://bradfrost.com/blog/post/atomic-web-design/)
- [Building the Facebook News Feed with Relay](https://facebook.github.io/react/blog/2015/03/19/building-the-facebook-news-feed-with-relay.html)


### 각주
> \* 이전 버전의 아티클에서 저는 smart and dumb 컴포넌트라고 불렀습니다. 그러나 이것은 프레젠테이션 컴포넌트에게 너무나 심한 표현이었습니다. 
> 그리고 가장 중요한점은 목적의 차이에 대해서 정확하게 설명할 수 없다는 것이었습니다. 저는 새로운 표현이 더 낫다고 생각했고 당신도 그랬으면 좋겠네요!
> 
> \*\* 이전 버전의 아티클에서 저는 프레젠테이션 컴포넌트가 프레젠테이션 컴포넌트만 포함해야 한다고 주장했었습니다. 
> 저는 더이상 다른 케이스를 생각해보지 않았습니다. 어떤 컴포넌트가 프레젠테이션 컴포넌트인지 컨테이너 컴포넌트인지는 그것의 구체적인 구현방법에 따라 달라지는 것입니다. 프레젠테이션 컴포넌트는 사이트 요청에 의한 변화가 없다면 컨테이너 컴포넌트로 변경이 가능해야 합니다. 그러므로 프레젠테이션 컴포넌트와 컨테이너 컴포넌트는 둘다 서로를 포함할 수 있습니다.
