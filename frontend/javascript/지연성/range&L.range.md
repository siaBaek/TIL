> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## range

- 숫자를 받아서 숫자 크기의 배열을 return
- range 함수를 통해 나온 배열의 모든 값들을 더하기

```javascript
const range = (l) => {
  let i = -1;
  let res = [];
  while (++i < l) {
    console.log(i, 'range'); // 0 1 2 3
    res.push(i);
  }
  return res;
};

const add = (a, b) => a + b;
var list = range(4);
console.log(list); // [0, 1, 2, 3]
console.log(reduce(add, list)); // 6
```

<br />
<br />

## 느긋한 L.range

```javascript
const L = {};
L.range = function* (l) {
  let i = -1;
  while (++i < l) {
    console.log(i, 'L.range');
    // 해당 값이 필요하기 전까지는 출력되지 않음
    yield i;
  }
};

var list = L.range(4);
console.log(list); // L.range {<suspended>}
console.log(reduce(add, list)); // 6
```

- L.range {\<suspended>}란, next() 메서드를 통해 값을 가져올 수 있는 이터레이터라는 의미이다.
- 결과가 같은 이유는 reduce가 이터러블을 받기 때문이다.

<br />
<br />

## range와 L.range의 차이

### range

- reduce에 list를 전달하기 전에 이미 range를 실행했을 때 list 변수에 담긴 값이 배열로 평가된 상태
- 즉, range를 실행했을 때 이미 모든 부분이 코드가 평가되며 값이 만들어진다.

### L.range

- L.range를 실행했을 때 list에 담긴 값이 배열로 평가되지 않은 상태
- 그렇다면 언제 처음 코드가 평가될까? 이터레이터 내부를 순회할 때마다 하나씩 값이 평가된다.
- next() 하기 전까지는 어떤 코드도 동작하지 않기 때문에 L.range 함수 코드 내부에서 콘솔을 찍어도 아무것도 출력되지 않았던 것이다.
- 즉, L.range를 실행하더라도, next()를 호출하기 전에는 어떤 코드도 평가되지 않는다.
- 평가가 완벽히 되지 않은 상태로 있다가, 실제로 reduce안에서 해당 값이 필요할 때 까지 기다렸다가 평가가 이루어져 값이 꺼내도록 되어있다.

### reduce의 구조

- 안쪽에서 이터레이터를 만들어주는 구조
- `for...of`에서 이터러블이 이터레이터가 되도록 안쪽에서 [Symbol.iterator]()를 해주는 과정이 있다.
- 따라서, range 함수의 경우 배열을 만들고, 그 배열을 이터레이터로 만들고 그 다음에 next() 하면서 순회하는 것이다.
- L.range 함수의 경우 실행됐을 때 이터레이터를 만들고, 그 이터레이터가 곧 자기 자신을 리턴하는 이터러블이고, 해당 함수를 실행하면 이미 만들어진 이터레이터를 리턴만 하고 순회하기 때문에 조금 더 효율적이라고 할 수 있다.

```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});
```

<br />
<br />

## 테스트

- 즉시 배열로 모든 값을 평가한 후, 해당 배열을 이터러블/이터레이터로 만들어서 reduce를 통해 add를 하는 `range` 코드
- 배열을 만들지 않고, 바로 이터레이터 만들어서 reduce를 통해 add를 하는 `L.range` 코드
- 두 코드가 어느 정도의 효율성을 가지고 있는지 성능을 테스트해보자

```javascript
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});

const range = (l) => {
  let i = -1;
  let res = [];
  while (++i < l) {
    console.log(i, 'range');
    res.push(i);
  }
  return res;
};

const L = {};
L.range = function* (l) {
  let i = -1;
  while (++i < l) {
    console.log(i, 'L.range');
    yield i;
  }
};

const add = (a, b) => a + b;

// time초만큼 f를 실행하는 함수
function test(name, time, f) {
  console.time(name);
  while (time--) f();
  console.timeEnd(name);
}

test('range', 10, () => reduce(add, range(1000000))); // 492.196...ms
test('L.range', 10, () => reduce(add, L.range(1000000))); // 256.035...ms
```
