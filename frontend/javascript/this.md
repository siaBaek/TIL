# 1. 상황에 따라 달라지는 this

## 전역 공간에서의 this 

- 전역 공간에서의 this는 전역 객체를 의미
- 브라우저 전역 객체 : window
- node.js 환경 전역 객체 : global

### 전역 공간에서만 발생하는 특이한 성질

- 전역 변수 선언 시 자바스크립트 엔진은 이를 전역 객체의 프로퍼티로도 할당 (변수이면서 객체의 프로퍼티)

```javascript
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

- 자바스크립트의 모든 변수는 실은 '특정 객체의 프로퍼티'로서 동작
- -> 특정 객체 : 실행 컨텍스트의 LexicalEnvironment. (전역 컨텍스트의 경우, LexicalEnvironment는 전역 객체 그대로 참조)

### 전역 변수와 전역 객체 프로퍼티

- 선언시 (같은 결과)

```javascript
window.a = 1;
console.log(a, window.a, this.a); // 1 1 1
```

- 선언 후 삭제시 (전혀 다른 결과)
- (delete 연산자는 window.delete와 같다고 이해하면 됨)

```javascript
var a = 1;
delete window.a; // false

var b = 2;
delete b; // false

window.c = 3;
delete window.c; // true

window.d = 4;
delete d; // true

console.log(a, window.a, this.a); // 1 1 1
console.log(b, window.b, this.b); // 2 2 2
console.log(c, window.c, this.c); // Uncaught ReferenceError : c is not defined
console.log(d, window.d, this.d); // Uncaught ReferenceError : d is not defined
```

- -> 전역 변수로 선언 시, 자바스크립트 엔진이 이를 자동으로 전역 객체 프로퍼티로 할당하며,
- 추가적으로 해당 프로퍼티의 configurable(변경 및 삭제 가능성) 속성을 false로 정의하기 때문에 변경 및 삭제를 할 수 없음

## 메서드로서 호출할 때 그 메서드 내부에서의 this

### 함수 vs. 메서드

- 앞에 객체가 명시되어 있으면 메서드, 그렇지 않으면 함수
- 독립성에 있어서 서로 차이가 있음
- 함수 : 그 자체로 독립적인 기능을 수행
- 메서드 : 자신을 호출한 대상 객체에 관한 동작을 수행

```javascript
var func = function(x) {
    console.log(this);
};
func();	// window {...}					-- 전역 객체 window

var obj = {
    method : func;
};
obj.mehtod(); // { method : f }				-- 자신을 호출한 객체 obj
```

- -> 익명함수는 그대로인데, 이를 변수에 담아 호출한 경우와 obj 객체의 프로퍼티에 할당해서 호출한 경우의 this가 다름

### 메서드 앞에 객체를 명시하는(=메서드를 호출하는) 2가지 방법

1. 객체명.메서드명();
2. 객체명\['메서드명'\]();

### 메서드 내부에서의 this

- 메서드 내부에서의 this는 호출한 주체에 대한 정보를 가리킨다
- 메서드로서 호출시 호출주체는 앞에 있는 객체

```javascript
var obj = {
    methodA : function() {
        console.log(this);
    },
    inner : {
        methodB : function() {
            console.log(this);
        };
    }
}
obj.methodA();		// { methodA : f, inner : {...} }	-- obj
obj.inner.methodB();	// { methodB : f }			-- obj.inner
```

## 함수로서 호출할 때 그 함수 내부에서의 this

### 함수 내부에서의 this

- 함수로서 호출 시 호출주체(객체)를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이라 호출주체 정보 알 수 없음
- 함수에서의 this는 지정하지 않으면 무조건 전역 객체

### 메서드 내부함수에서의 this

- 함수로서의 호출인지 메서드로서의 호출인지만 파악하면 됨 (함수인데 지정하지 않았다면 this, 메서드라면 호출한 객체)

```javascript
var obj1 = {
    outer : function() {
        console.log(this);
        var innerFunc = function() {
            console.log(this);
        }
        innerFunc();	// (2)
        var obj2 = {
            innerMethod : innerFunc;
        };
        obj2.innerMethod();	// (3)
    }
};
obj1.outer();	// (1)
```

- (1) outer() 앞에 obj1.이 있기 때문에 메서드로서의 호출
- { outer : f } ( obj1 바인딩) 출력
- (2) innerFunc() 앞에 객체가 없기 때문에 함수로서의 호출
- window {...} (전역 객체 바인딩) 출력
- (3) innerMethod() 앞에 obj2.가 있기 때문에 메서드로서의 호출
- { innerMethod : f } ( obj2 바인딩 ) 출력

### 메서드의 내부함수에서의 this를 우회하는 방법 (ES5)

- 호출 주체 없을 경우 자동으로 전역 객체를 바인딩하지 않고 호출 당시 주변 환경의 this를 상속받고 싶을 때는 변수를 활용

```javascript
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc1 = function () {
      console.log(this);
    };
    innerFunc1(); // window {...}

    var self = this; // 변수 선언 : 상위 스코프의 this를 저장해서 내부함수에서 사용
    var innerFunc2 = function () {
      console.log(this);
    };
    innerFunc2(); // { outer : f }
  },
};
obj.outer(); // { outer : f }
```

### this를 바인딩하지 않는 함수 (ES6 화살표 함수)

- 함수 내부에서 this가 전역 객체를 바라보는 문제를 보완하기 위해 등장
- 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어 상위 스코프의 this를 그대로 활용 가능

```javascript
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = () => {
      console.log(this);
    };
    innerFunc(); // { outer : f }
  },
};
obj.outer(); // { outer : f }
```

## 콜백 함수 호출 시 그 함수 내부에서의 this

### 콜백함수

- 함수A의 제어권을 다른 함수B(또는 메서드)에게 넘겨주는 경우에서의 함수A를 콜백함수라고 함
- 함수B의 로직에 따라 함수A를 실행하여 this 역시 함수B 내부 로직에 따라 결정됨
- 함수에서의 this는 기본적으로 전역 객체를 참조하지만,
- 제어권을 받은 함수에서 콜백함수에게 별도로 this가 될 대상을 지정하면 그 대상을 참조함

```javascript
// (1)
setTimeout(function () {
  console.log(this);
}, 300);

// (2)
[1, 2, 3, 4, 5].forEach(function (x) {
  consoe.log(this.x);
});

// (3)
document.body.innerText += '<button id="a">클릭</button>';
document.body.querySelect('#a').addEventListener('click', function (e) {
  console.log(this, e);
});
```

- (1)(2) this 대상 지정하지 않음 -> 전역 객체인 window {...} 출력
- (1) 0.3초 후 콘솔창에 window {...} 출력
- (2) window{...} 와 각 요소를 한 번씩 총 5번 출력
- (3) 앞서 지정한 버튼 엘리먼트와 클릭 이벤트에 관한 정보가 담긴 객체 출력
- .addEventLister() 메서드는 기본적으로 콜백함수 호출시 자신의 this를 상속하는 성질이 있음

<br />

- 따라서 호출된 콜백함수 내부의 this는 .addEventListener()의 this인 document.body.querySelect('#a')인 버튼을 가리킴
- -> 콜백함수의 제어권을 가지는 함수(메서드)가 콜백함수에서의 this를 무엇으로 할지 결정하고
- 특별히 정의하지 않은 경우 기본적 함수의 this 처럼 전역 객체를 바라봄

## 생성자 함수 내부에서의 this

### 생성자 함수

- 공통된 성질을 지니는 객체들을 생성하는데 사용하는 함수
- 자바스크립트에서 생성자는 클래스(class)를 의미하고, 클래스를 통해 만든 객체는 인스턴스(instance)를 의미함
- 생성자는 구체적인 인스턴스를 만들기위한 일종의 틀이라고 생각하면 됨
- ( 미리 준비된 공통 속성에 구체적인 인스턴스의 개성을 더해 개별 인스턴스를 만드는 것임 )
- 자바스크립트는 함수에 생성자 역할을 함께 부여해서, new 명령어로 함수 호출하면 생성자로 동작하게 됨
- 생성자 내부 this는 새로 만들 구체적인 인스턴스 자신을 가리킴

### 구체적인 인스턴스 생성과정

- new 명령어로 함수 호출 시 생성자의 prototype 프로퍼티를 참조하는 \_\_proto\_\_라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여함

```javascript
var Cat = function (name, age) {
  this.bark = '야옹';
  this.name = name;
  this.age = age;
}; // 프로퍼티에 각 값을 대입

var choco = new Cat('초코', 7); // 생성자 함수 내부에서의 this는 생성되는 choco 인스턴스
var nabi = new Cat('나비', 5); // 생성자 함수 내부에서의 this는 생성되는 nabi 인스턴스

console.log(choco, nabi);

/*
결과
Cat { bark:'야옹', name:'초코', age:7 }
Cat { bark:'야옹', name:'나비', age:5 }
*/
```

<br />
<br />

# 2. 명시적으로 this를 바인딩하는 방법

- this에 별도의 대상을 바인딩하는 방법

## call 메서드

```javascript
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

- 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령
- call 메서드의 첫 번째 인자를 this로 바인딩하고 이후 인자들을 호출할 함수의 매개변수로 함
- 임의의 객체를 this로 지정할 수 있음

```javascript
var func = function (a, b, c) {
  console.log(this, a, b, c);
};

func(1, 2, 3); // Window {...} 1 2 3
func.call({x: 1}, 4, 5, 6); // { x: 1 } 4 5 6
```

```javascript
var obj = {
  a: 1,
  method: function (x, y) {
    console.log(this.a, x, y);
  },
};

obj.method(2, 3); // 1 2 3
obj.method.call({a: 4}, 5, 6); // 4 5 6
```

## apply 메서드

```javascript
Function.prototype.apply(thisArg[, argsArray])
```

- 기능적으로 call 메서드와 동일
- call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하고,
  apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정함

```javascript
var func = function (a, b, c) {
  console.log(this, a, b, c);
};
func.apply({x: 1}, [4, 5, 6]); // { x: 1 } 4 5 6

var obj = {
  a: 1,
  method: function (X, y) {
    console.log(this.a, x, y);
  },
};
obj.method.apply({a: 4}, [5, 6]); // 4 5 6
```

## call / apply 메서드의 활용

### 유사배열객체(array-like object)에 배열 메서드를 적용

1. 유사배열객체에 배열 메서드를 적용

- 객체에는 배열 메서드를 직접 적용할 수 없으므로, 키가 0 또는 양의 정수인 프로퍼티가 존재하고 length 프로퍼티의 값이 0 또는 양의 정수인 객체(유사배열객체)의 경우 call / apply 메서드를 이용해 배열 메서드를 차용할 수 있음

```javascript
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
};
Array.prototype.push.call(obj, 'd'); // 배열 메서드인 push를 적용
console.log(obj); // { 0: 'a', 1: 'b', 2: 'c', 3: 'd', length: 4 }

var arr = Array.prototype.slice.call(obj); // slice - 객체를 배열로 전환
console.log(arr); // ['a', 'b', 'c', 'd']
```

- slice 메서드 : 시작 인덱스값과 마지막 인덱스값을 받아 시작값부터 마지막값의 앞 부분까지의 배열 요소를 추출하는 메서드
- ( 매개변수를 아무것도 넘기지 않으면 그냥 원본 배열의 얕은 복사본을 반환 )
- -> call 메서드를 이용해 원본 유사배열객체의 얕은 복사를 수행한 것인데 slice 메서드가 배열 메서드라 복사본은 배열로 반환

2. arguments, NodeList에 배열 메서드를 적용

- argument 객체도 유사배열객체이므로 배열로 전환해서 활용 가능
- NodeList(querySelectorAll, getElementByClassName 등)도 마찬가지

```javascript
function a() {
  var argv = Array.prototype.slice.call(arguments);
  argv.forEach(function (arg) {
    console.log(arg);
  });
}
a(1, 2, 3);

document.body.innerHTML = '<div>a</dov><div>b</div><div>c</div>';
var nodeList = document.querySelectAll('div');
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function (node) {
  console.log(node);
});
```

3. 문자열에 배열 메서드 적용 예시

- 배열처럼 인덱스와 length 프로퍼티를 지니는 문자열도 배열 메서드 적용 가능
- 단, 문자열의 경우 length 프로퍼티가 읽기 전용이기 때문에
- 원본 문자열에 변경을 가하는 메서드는 에러를 던지며,
- concat처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없음

```javascript
var str = 'abc def';

Array.prototype.push.call(str, ', pushed string');
// Error: Cannot assign to read only property 'length' of object [object String]

Array.prototype.concat.call(str, 'string'); // [string {"abc def"}, "string"]

Array.prototype.every.call(str, function (char) {
  return char !== ' ';
}); // false
Array.prototype.some.call(str, function (char) {
  return char === ' ';
}); // true

var newArr = Array.prototype.map.call(str, function (char) {
  return char + '!';
});
console.log(newArr); // ['a!', 'b!', 'c!', ' !', 'd!', 'e!', 'f!']

var newStr = Array.prototype.reduce.apply(str, [
  function (string, char, i) {
    return string + char + i;
  },
  '',
]);
console.log(newStr); // "a0b1c2 3d4e5f6"
```

4. Array.from 메서드 (ES6)

- call / apply를 이용해 형변환하는 것은 this를 원하는 값으로 지정해서 호출한다는 본래의 메서드의 의도와는 다소 동떨어짐
- 코드만 봐서는 어떤 의도인지 파악하기 쉽지 않음
- -> ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터 타입을 배열로 전환하는 Array.from 메서드를 새로 도입

```javascript
var obj = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
};
var arr = Array.from(obj);
console.log(arr); // ['a', 'b', 'c']
```

### 생성자 내부에서 다른 생성자를 호출

- 생성자 내부에 다른 생성자와 공통된 내용이 있을 경우 call / apply를 이용해 다른 생성자를 호출하면 반복 줄일 수 있음

```javascript
function Person(name, gender) {
  this.name = name;
  this.gender = gender;
}
function Student(name, gender, school) {
  Person.call(this, name, gender);
  this.school = school;
}
function Employee(name, gender, company) {
  Person.apply(this, [name, gender]);
  this.company = company;
}
var sa = new Student('시아', 'female', '단국대');
var hj = new Employee('형준', 'male', '카카오');
```

### 여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply 활용

예) 배열에서 최대/최솟값을 구하기

1. apply 활용하기

```javascript
var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);
console.log(max, min); // 45 3
```

- 만약 apply를 사용하지 않는다면 코드가 불필요하게 길고 가독성도 떨어질 것임

```javascript
var numbers = [10, 20, 3, 16, 45];
var max = (min = numbers[0]);
numbers.forEach(function (number) {
  if (number > max) {
    max = number;
  }
  if (number < min) {
    min = number;
  }
});
console.log(max, min); // 45 3
```

1. ES6의 펼치기 연산자(spread operator) 활용

- apply를 적용하는 것보다 더욱 간편하게 작성 가능

```javascript
const numbers = [10, 20, 3, 16, 45];
const max = Math.max(...numbers);
const min = Math.min(...numbers);
console.log(max, min); // 45 3
```

### call / apply 특징

- call / apply 메서드는 명시적으로 별도의 this를 바인딩하면서 함수/메서드를 실행하는 훌륭한 방법
- 오히려 이로 인해 this를 예측하기 어렵게 만들어 코드 해석 방해한다는 단점
- 그럼에도 불구하고 ES5 이하에선 마땅한 대안이 없어 매우 광범위하게 활용

## bind 메서드

- call과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드
- 다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달해던 인수들의 뒤에 이어서 등록

### bind 메서드의 2가지 목적

- 함수에 this를 미리 적용하는 것
- 부분 적용 함수를 구현

### this 지정과 부분 적용 함수 구현

```javascript
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};
func(1, 2, 3, 4);

var bindFunc1 = func.bind({x: 1});
bindFunc1(5, 6, 7, 8); // {x: 1} 5 6 7 8

var bindFunc2 = func.binc({x: 1}, 4, 5);
bindFunc2(6, 7); // { x: 1 } 4 5 6 7
bindFunc2(8, 9); // { x: 1 } 4 5 8 9
```

### name 프로퍼티

- bind 메서드를 적용해 새로 만든 함수가 가지는 특징
- name 프로퍼티에 동사 bind의 수동태인 'bound'라는 접두어가 붙음
- 기존의 call / apply 보다 코드 추적이 수월해짐

```javascript
var func = function (a, b, c, d) {
  console.log(this, a, b, c, d);
};
var bindFunc = func.bind({x: 1}, 4, 5);
console.log(func.name); // func
console.log(bindFunc.name); // bound func
```

### 상위 컨텍스트의 this를 내부함수나 콜백 함수에 전달하기

- self 등의 변수를 활용한 우회법보다 call, apply, bind 메서드를 이용하면 더 깔끔하게 처리 가능

1. 내부함수에 this 전달

```javascript
// call 메서드 이용
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this);
    };
    innerFunc.call(this);
  },
};
obj.outer();
```

```javascript
// bind 메서드 이용
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = function () {
      console.log(this);
    }.bind(this);
    innerFunc();
  },
};
obj.outer();
```

2. 콜백 함수 내에서의 this에 관여하는 함수/메서드

```javascript
var obj = {
  logThis: function () {
    console.log(this);
  },
  logThisLater1: function () {
    setTimeout(this.logThis, 500);
  },
  logThisLater2: function () {
    setTimeout(this.logThis.bind(this), 1000);
  },
};
obj.logThisLater1(); // Window { ... }
obj.logThisLater2(); // obj { logThis: f, logThisLater1: f, logThisLater2: f }
```

## 화살표 함수의 예외사항

- 별도의 변수로 this를 우회하거나 call, apply, bind 메서드를 적용할 필요가 없음
- 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외되어 함수 내부에 this가 아예 없으며,
- 접근하고자 하면 스코프체인상 가장 가까운 this에 접근함

```javascript
var obj = {
  outer: function () {
    console.log(this);
    var innerFunc = () => {
      console.log(this);
    };
    innerFunc(); // { outer: f }
  },
};
obj.outer();
```

## 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)

- 콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this를 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있음
- -> 이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있음
- 보통 여러 내부 요소에 대해 같은 동작을 반복 수행하는 배열 메서드가 이런 형태 (ES6의 Set, Map 등에서도 일부 존재)

```javascript
var report = {
  sum: 0,
  coun: 0,
  add: function () {
    var args = Array.prototype.slice.call(arguments);
    args.forEach(function (entry) {
      this.sum += entry;
      ++this.count;
    }, this);
  },
  average: function () {
    return this.sum / this.count;
  },
};
report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); // 240 3 80
```

- -> 콜백 함수 내부에서의 this는 forEach 함수의 두 번째 인자로 전달해준 this가 바인딩됨

### 콜백 함수와 함께 thisArg를 인자로 받는 메서드 - 메서드명(callback\[, thisArg\])

- forEach, map, filter, some, every, find, findIndex, flatMap, from
- Set.prototype.forEach
- Map.prototype.forEach
