---
title: '[javascript] this는 어렵지 않습니다.'
date: 2018-03-12 01:42:51
tags:
- javascript
- this
- arrow function
- bind
- subroutine
---
this는 어렵지 않습니다.

this를 어렴풋이 알고는 있지만, 누가 물어봤을때 제대로 대답해 줄수 있도록 정리해보겠습니다. 많은 개발자들이 javascript의 this를 혼란스러워합니다. 사실 개념 자체가 어렵진 않습니다. 다만, 다른 프로그래밍 언어들과 사용법에 차이가 있을 뿐이죠. 언어마다 조금은 차이가 있겠지만 대표적으로 JAVA같은 객체지향 언어에서의 this는 클래스 인스턴스의 레퍼런스 변수입니다. 하지만 javascript에서 this는 전혀 다른 의미를 가집니다. 개발을 시작하고 처음으로 javascript를 접한 개발자라면 조금 덜 혼란스러울지도 모르겠지만, 많은 개발자들이 C, C++, Java, python 등의 언어를 먼저 배운 뒤 javascript를 접하는 케이스가 많습니다. 또한, Jquery 등의 라이브러리에 의존하는 경향 때문에 언어 자체의 문법이나 특성의 이해보다는 사용법만 습득하기도 하죠. 이런 경우, javascript의 this가 충분히 혼란스러울수 있을것 같습니다. 

### this는 현재 실행 문맥이다
`실행문맥`이란 말은 호출자가 누구냐는 것과 같습니다.
```
alert(this === window); // true, 호출자는 window

const caller = {
  f: function() {
    alert(this === window);
  } 
}
caller.f(); // false, 호출자는 caller 객체
```
첫번째는 함수 호출, 두번째는 메소드 호출이라고 말하는데 이런 구분이 괜한 혼란을 야기합니다. 첫번째 alert도 따지고보면 `window.alert()`과 동일하기 때문에 window 객체의 메소드 호출이라봐도 무방합니다. 다만, `strict-mode`에서는 전역 객체냐 일반 객체냐에 따라 함수내부에 this의 결과가 다르다는 차이는 있죠. 그러나 이 문제 또한 window를 함수 호출 앞에 붙여주면 해결됩니다.
```
function nonStrictMode() {
  return this;
}

function strictMode() {
  'use strict'
  return this;
}

console.log(nonStrictMode()); // window
console.log(strictMode()); // undefined
console.log(window.stricMode()); // window
```

### 생성자 함수 / 객체에서는 어떻게 쓰이나?
생성자는 new로 객체를 만들어 사용하는 방식입니다. 객체지향 언어에서 일반적으로 객체를 만들 때 쓰이는 문법과 동일하죠. 가리키는 대상 또한 객체지향 언어의 `this`와 같기 때문에 이해하기가 한결 수월합니다.
```
function NewObject(name, color) {
  this.name = name;
  this.color = color;
  this.isWindow = function() {
    return this === window;
  }
}

const newObj = new NewObject('nana', 'yellow');
console.log(newObj.name); // nana
console.log(newObj.color); // yellow
console.log(newObj.isWindow()); // false

const newObj2 = new NewObject('didi', 'red');
console.log(newObj2.name); // didi
console.log(newObj2.color); // red
console.log(newObj2.isWindow()); // false
```
new 키워드로 새로운 객체를 생성했을 경우 생성자 함수 내의 this는 new를 통해 만들어진 새로운 변수가 됩니다. `newObj`, `newObj2`는 같은 생성자 함수로 만들어진 객체이지만 완전히 별도의 객체이기 때문에 각 객체의 속성들은 서로 관련이 없습니다. 만약 new 키워드를 빼먹으면 어떻게 될까요? 
```
const withoutNew = NewObject('nana', 'yellow');
console.log(withoutNew.name); // error
console.log(withoutNew.color); // error
console.log(withoutNew.isWindow()); // error
```
new 키워드가 없으면 일반적인 함수 실행과 동일하게 동작하므로, `NewObject` 함수내의 this는 `window` 객체가 됩니다. 하지만 `withoutNew`가 함수 실행의 결과값이 할당되므로 각 property를 가져올 수 없습니다. 
그렇다면, 생성자 함수가 아닌 일반 객체에서는 어떨까요?
```
const person = {
  name: 'john',
  age: 15000,
  nickname: 'man from earth',
  getName: function () {
    return this.name;
  }
}
console.log(person.getName()); // john

const otherPerson = person;
otherPerson.name = 'chris';
console.log(person.getName()); // chris
console.log(otherPerson.getName()); // chris
```
생성자 함수와 크게 다르지 않습니다. 한가지 눈여겨 볼 점은 `otherPerson.name`을 `chris`로 설정한 뒤 person.getName() 호출하면 그 결과는 `chris`입니다. 그 이유는 otherPerson은 person의 레퍼런스 변수이므로 하나(otherPerson)를 변경하면 다른 하나(person)도 변경됩니다. 이를 피하기 위해서는 `Object.assign()`메서드(ES6 지원)를 이용하여 완전히 별도의 객체로 만들어야 합니다.
```
const person = {
  name: 'john',
  age: 15000,
  nickname: 'man from earth',
  getName: function () {
    return this.name;
  }
}
const newPerson = Object.assign({}, person);
newPerson.name = 'chris';
console.log(person.getName()); // john
console.log(newPerson.getName()); // chris
```

### bind, arrow function
이번에는 생성자 함수 안에서 또 다른 함수가 있는 경우를 살펴보겠습니다.
```
function Family(firstName) {
  this.firstName = firstName;
  const names = ['bill', 'mark', 'steve'];
  names.map(function(lastName, index) {
    console.log(lastName + ' ' + this.firstName);   
	console.log(this);      
  });
}
const kims = new Family('kim');
// bill undefined
// window
// mark undefined
// window
// steve undefined
// window
```
`Family`라는 생성자 함수 안에서 `map` 메서드를 호출합니다. map 메서드의 인자는 value와 index를 인자로 가지는 새로운 함수입니다. 이를 `서브루틴`이라 부르겠습니다. 서브루틴이 특별히 다른 개념은 아닙니다. 자바스크립트에서 함수의 의미가 다양하기 때문에 단지 메서드가 아닌 함수와 구분하기 위한 용도로 서브루틴이라는 단어를 사용합니다. 

이 서브루틴에서는 lastName들을 담은 `names` 배열의 map 메서드를 이용하여 lastName과 this의 firstName을 같이 출력하고자 합니다. 하지만 막상 실행을 해보면 예상과 다르게 출력됩니다. kim이 출력될 위치에 `undefined`가 출력되었습나다. 이는 map의 서브루틴에서 this를 사용하는 것이 문제였습니다. this가 실행 문맥이라고 했던것을 상기해보면 undefined가 출력되는 이유를 짐작해볼 수 있습니다. map 메서드의 서브루틴은 호출될때 map의 context(this)로 바인드 되지 않습니다. 바인드 되지 않았다는 것은 실행문맥이 전역이라는 것이고 실행문맥이 전역이란 말은 (비엄격모드에서) 서브루틴 내 this가 `window`라는 것입니다.

비슷한 현상을 다른 예제에서 살펴보겠습니다. 아래 함수를 실행시키면 innerFunc안의 this는 window가 출력됩니다. 
```
const testObj = {
  outerFunc:  function() {
    function innerFunc() {
        console.log(this); // window
    }
    innerFunc();
  }
}
testObj.outerFunc();   
```
outherFunc가 외부에서 실행(testObj.outerFunc())되면 this는 testObj입니다. 그리고 outerFunc 내부에서 innerFunc가 호출할때는 그 어떤 문맥도 지정하지(바인드되지) 않았습니다. 전역 context(window)에서 실행되었다는 것이죠. 이게 바로 (비엄격모드에서) innerFunc의 this가 window가 되는 이유 입니다.

다시 이전의 생성자 함수(Family)로 돌아갑니다. map 메서드의 서브루틴에서 this가 window가 된다는 것은 위에서 이미 설명했습니다. 하지만, 생성자 함수 내의 특정 변수를 서브루틴 내에서 사용할 수도 있습니다. 이 때, 실행문맥(this)을 Family로 지정하려면 간단하게는 별도의 상수(const)를 지정하면 됩니다.
```
function Family(firstName) {
    this.firstName = firstName;
    const names = ['bill', 'mark', 'steve'];
    const that = this;
    names.map(function(value, index) {
        console.log(value + ' ' + that.firstName); 
    })
}
const kims = new Family('kim');
// bill kim
// mark kim
// steve kim
```
문제 없이 이름들이 출력됩니다. 하지만, 항상 `that`이라는 상수를 만들어주면 귀찮습니다. 또한, 만에 하나 실수로 빼먹기라도 하면 어마어마한 문제가 발생할지도 모릅니다. 혹은 서브루틴 안에서 또다른 서브루틴을 사용할 수도 있습니다. 그 때는 `anotherThat`을 만들어야 할까요? 이 문제를 해결하기 위해서 `bind`라는 메서드를 사용합니다.
```
function Family(firstName) {
    this.firstName = firstName;
    const names = ['bill', 'mark', 'steve'];
    names.map(function(value, index) {
        console.log(value + ' ' + this.firstName); 
    }.bind(this));
}
const kims = new Family('kim');
```
that을 쓸때보다는 깔끔해졌습니다. 하지만 `.bind(this)`도 항상 붙여줘야한다는 문제는 여전히 남아 있습니다. 이제 `arrow function`이 나올때가 된것 같네요.
```
function Family(firstName) {
    this.firstName = firstName;
    const names = ['bill', 'mark', 'steve'];
    
    names.map((value, index) => {
        console.log(value + ' ' + this.firstName); 
    });
}
const kims = new Family('kim');
```
이제 that도 없고, bind도 없습니다. 함수의 형태만 바꿔주면 모든게 해결됩니다. 그럼 일반 함수형태에서 arrow 함수를 사용했을때 어떤 차이가 있을까요? arrow 함수 또한 ES6에서만 지원하기 때문에 babel 사이트에서 변환해보겠습니다.
```
"use strict";

function Family(firstName) {
  var _this = this;

  this.firstName = firstName;
  var names = ["bill", "mark", "steve"];
  names.map(function(value, index) {
    console.log(value + " " + _this.firstName);
  });
}
var kims = new Family("kim");
```
that을 사용했을 때와 동일한 방법으로 트랜스파일 되네요. 미리 내부에서만 사용할 변수 `_this`를 만들어 두고, this를 할당합니다. 그리고 `_this`를 사용하여 firstName을 가져옵니다. arrow 함수는 호출 대상에 따라 실행문맥이 결정되는 것이 아닙니다. 

### 결론
this는 어렵지 않습니다. 하지만, 타 언어와 다른 방식으로 사용되기에 주의해서 사용할 필요가 있습니다. 한가지만 기억하자면, this는 누가 호출했느냐에 따라 결정된다는 것입니다. ES6 문법을 사용하면 this를 사용할때 문제점을 완화할 수 있습니다. 예를들어, 서브루틴 내에서 바깥의 this를 사용하려고 할때는 arrow function을 이용하면 간단하게 해결할 수 있습니다.

참고 자료
* https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this
* https://developer.mozilla.org/ko/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript
* http://webframeworks.kr/tutorials/translate/explanation-of-this-in-javascript-1/
* https://gomugom.github.io/is-class-only-a-syntactic-sugar/
* http://webframeworks.kr/tutorials/translate/arrow-function/
