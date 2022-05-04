> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## map

- 함수형 프로그래밍에서는 함수가 `인자`와 `리턴 값`으로 소통하는 것을 권장한다.
- `console.log(names)`처럼 names를 직접적인 변화를 일으키는 다른 함수/메서드에 보내는 것이 아니라, 결과를 return해서 return 값을 이후에 사용하게 된다.
- 문장으로 작성할 땐 `a.name`처럼 어떤 값을 수집할지 개발자가 직접 작성했었지만, map 함수는 함수를 받아서 이 부분을 `추상화`한다. (함수를 인자로 받았기 때문에 고차함수)
- [map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

```javascript
// 출석부에 있는 이름이나 나이를 따로 배열에 담는 로직을 종종 사용
const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 명령형 프로그래밍
let names = [];
for (const a of attendance) {
  names.push(a.name);
}
console.log(names); // (5) ['john', 'paul', 'james', 'sasha', 'susan']

let ages = [];
for (const a of attendance) {
  ages.push(a.age);
}
console.log(ages); // (5) [20, 25, 20, 30, 27]

// 함수형 프로그래밍
const map = (f, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
};

console.log(map((p) => p.name, attendance)); // (5) ['john', 'paul', 'james', 'sasha', 'susan']
console.log(map((p) => p.age, attendance)); // (5) [20, 25, 20, 30, 27]
```

<br />

### 이터러블 프로토콜을 따른 map의 높은 다형성

- map 함수는 Array 뿐 아니라 이터러블 프로토콜을 따르는 많은 함수들을 다 사용할 수 있게 된다.
- 프로토타입/클래스 기반으로 어떤 뿌리를 가진 값만 어떤 함수를 사용할 수 있게 하는 것 보다 훨씬 유연하고 다형성이 높다.

```javascript
document.querySelectAll('*').map((el) => el.nodeName); // Array 형태지만 map함수를 사용할 수 없다 (~map is not a function)
// 이유는 Array를 상속받은 객체가 아니기 때문에 prototype에 map함수가 구현되어 있지 않다
console.log(map((el) => el.nodeName, document.querySelectorAll('*'))); // 앞서 만든 map함수 사용시 에러 발생하지 않는다
// document.querySelectorAll('*'))가 이터러블 프로토콜을 따르고 있기 때문에 가능하다
const iter = document.querySelectorAll('*')[Symbol.iterator]();
console.log(iter); // Array Iterator {}
iter.next(); //{value: ..., done: false}
```

- 문장으로 이루어진 제너레이터 함수의 결과들에 대해서도 map을 사용할 수 있다.

```javascript
function* gen() {
  yield 2;
  if (false) yield 3;
  yield 4;
}
console.log(map((a) => a * a, gen())); // (2) [4, 16]
```

- map 함수와 달리 앞서 살펴본 key value 쌍을 표현하는 map (이터러블)
- 맵 객체 생성: `let m = new Map()`과 같이 맵 객체를 만드는데, map함수를 사용해서 안쪽 값이 바뀐 맵 객체를 만들 수 있다.

```javascript
let m = new Map();
m.set('a', 10);
m.set('b', 20);
const it = m[Symbol.iterator]();
it.next(); // {value: Array(2), done: false}
console.log(map(([k, a]) => [k, a * 2], m)); // (2) [ (2) ['a', 20], (2) ['b', 40] ]
console.log(new Map(map(([k, a]) => [k, a * 2], m))); // Map(2) {'a' => 20, 'b' => 40}
```
