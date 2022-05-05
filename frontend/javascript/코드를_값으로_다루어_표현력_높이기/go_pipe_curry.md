> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

- 함수형 프로그래밍에서는 코드를 값으로 다루는 아이디어를 많이 사용
- 코드를 값으로 다룰 수 있기 때문에 어떤 함수가 코드인 함수를 받아서 평가하는 시점을 원하는대로 다룰 수 있어서 코드의 표현력을 높일 수 있다.

<br />

#### [map,filter,reduce 중첩사용](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/map_filter_reduce/map%2Bfilter%2Breduce_%EC%A4%91%EC%B2%A9%EC%82%AC%EC%9A%A9.md)의 함수 중첩을 없애고 읽기 편하게 코드 변경해보기

## go

- go 함수는 인자가 list 형태로 들어온다 -> `spread operator`를 이용하여 인자를 설정해주고 reduce의 두 번째 인자로 list 형태로 전달한다.
- 첫 번째 인자를 다음 인자인 함수에게 전달하고, 그 결과를 또 다음 함수에게 전달하는, 인자를 재귀적으로 축약해나가는 로직 -> [reduce](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/map_filter_reduce/reduce.md)
- reduce의 2번째 인자로 args를 적으면, 함수 호출 시 첫 번째 인자가 초기값으로 설정되게 된다.

```javascript
const reduce = (f, acc, iter) => {
  // 초기값 생략 시(3번째 인자가 없을 시) 첫 번째 value를 초기값으로
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const go = (...args) => reduce((a, f) => f(a), args);

go(
  0,
  (a) => a + 1,
  (a) => a + 10,
  (a) => a + 100,
  console.log
);
// 111
```

<br />
<br />

## pipe

- 함수를 리턴하는 함수
- [go 함수](https://github.com/siaBaek/TIL/tree/main/frontend/javascript/코드를_값으로_다루어_표현력_높이기/go.md)는 즉시 평가하는 함수라면, pipe 함수는 함수가 나열된 합성된 함수를 만드는 함수

<br />

- 새로운 go 함수 형태의 코드를 pipe가 리턴하는 함수에 만들어 주면 된다.
- f 함수 호출 시가 받을 인자인 a를 ...as로 수정하여 여러 개의 인자를 받는다.
- go 시작하는 부분을 add(0, 1) 처럼 f 함수에 들어있는 함수 중 첫 번째 함수에는 인자를 펼쳐서 전달을 하고, 그 다음 함수들이 실행되도록 하면 된다.
- pipe 함수에 전달되는 인자를 rest 문법으로 (f, ...fs)처럼 설정하면, 첫 번째 함수만 꺼내고 나머지 함수들을 뒤에 넣을 수 있다.
- pipe 함수 내부의 go 함수 실행 시 go(f(...as), ...fs)처럼 실행해주면 go 호출 코드와 동일한 표현이 된다.

```javascript
const reduce = (f, acc, iter) => {
  // 초기값 생략 시(3번째 인자가 없을 시) 첫 번째 value를 초기값으로
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const go = (...args) => reduce((a, f) => f(a), args);
const pipe =
  (f, ...fs) =>
  (...as) =>
    go(f(...as), ...fs);

// 시작하는 인자가 2개여야할 때는 첫 번째 인자를 add(0, 1) 처럼 만들 수 있다
const add = (a, b) => a + b;
go(
  add(0, 1),
  (a) => a + 10,
  (a) => a + 100,
  console.log
);

// 시작하는 인자가 2개여야할 때는 호출 시 f(add(0,1))처럼 작성해야하는 것은 조금 아쉬움
// 첫 번째 인자를 2개 받을 수 있도록 수정
const f = pipe(
  (a, b) => a + b,
  (a) => a + 10,
  (a) => a + 100
);

console.log(f(0, 1)); // 111
```

<br />
<br />

## go를 사용하여 읽기 좋은 코드로 만들기

- [map, filter, reduce 중첩사용](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/map_filter_reduce/map%2Bfilter%2Breduce_%EC%A4%91%EC%B2%A9%EC%82%AC%EC%9A%A9.md)에 있는 코드를 go를 사용하여 읽기 좋은 코드로 만들 수 있다.
- 함수를 위에서 아래로 (왼쪽에서 오른쪽으로) 평가하며 연속적으로 함수를 실행하고 이전에 실행된 함수의 결과를 다음 함수에게 전달하는 go 함수를 만들었기 때문에, 코드를 바꿔줄 수 있다.

```javascript
const map = (f, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
};

const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};

const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 기존 코드
const add = (a, b) => a + b;
console.log(
  reduce(
    add,
    map(
      (a) => a.age,
      filter((a) => a.age < 25, attendance)
    )
  )
); // 40

// go 이용하기
const go = (...args) => reduce((a, f) => f(a), args);

go(
  attendance,
  (attendance) => filter((a) => a.age < 25, attendance),
  (attendance) => map((a) => a.age, attendance),
  (ages) => reduce(add, ages),
  console.log
); // 40
```

- 코드 양은 많고 덜 간결하게 보이지만, 읽기 편해졌다.
- 데이터의 처리 과정이 한 눈에 보인다.

<br />
<br />

## curry

- 함수를 값으로 다루면서 받아둔 함수를 원하는 시점에 평가시키는 함수
- 함수를 받아서 함수를 리턴하고,
- 리턴된 함수가 실행되었을 때의 리턴 값을 인자로 받아서,
- 들어온 인자의 개수가 2개 이상이라면 받아둔 함수를 즉시 실행하고,
- 2개 미만이라면 다시 함수를 리턴해 이후에 받을 인자들을 합쳐서 실행하는 함수

```javascript
// 인자가 2개 이상 전달 되었을 때 = 2번째 rest 인자가 length가 있을 때
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

// curry로 감싸면 함수를 전달해서 함수가 즉시 리턴
const mult = curry((a, b) => a * b);
console.log(mult); // (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._)

// 인자를 1개만 전달했을 경우
console.log(mult(1)); // (..._) => f(a, ..._)
// 나머지 인자를 더 전달했을 때 받아둔 함수에게 받아둔 인자와 지금받은 인자를 전달
console.log(mult(1)(2)); // 2

// 인자를 2개 전달했을 경우
console.log(mult(1, 2)); // 2

// 이런 식으로 사용할 수 있는 패턴이다
const mult3 = mult(3);
console.log(mult3(10)); // 30
console.log(mult3(5)); // 15
console.log(mult3(3)); // 9
```

## go + curry

- [여기](#go를-사용하여-읽기-좋은-코드로-만들기) 있는 코드를 go+curry로 조금 더 간결하게 만들어 볼 수 있다.

```javascript
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

// filter, map, reduce 모두 curry 적용
// 이제 인자를 하나만 받으면 이후 인자를 기다리는 함수를 리턴하도록 되었다
const map = curry((f, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
});

const filter = curry((f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
});

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

const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 기존 코드
const add = (a, b) => a + b;
const go = (...args) => reduce((a, f) => f(a), args);

go(
  attendance,
  (attendance) => filter((a) => a.age < 25, attendance),
  (attendance) => map((a) => a.age, attendance),
  (ages) => reduce(add, ages),
  console.log
);

// go+curry

go(
  attendance,
  (attendance) => filter((a) => a.age < 25)(attendance),
  (attendance) => map((a) => a.age)(attendance),
  (ages) => reduce(add)(ages),
  console.log
);

// attendance를 받아서 filter((a) => a.age < 25)함수에 그대로 attendance를 전달한다는 것은,
// go의 두 번째 인자에 들어오는 함수가 attendance를 받는다는 말이고,
// attendance를 받는 filter((a) => a.age < 25) 코드를 평가한 결과가
// filter((a) => a.age < 25) 와 동일하다는 것

go(
  attendance,
  filter((a) => a.age < 25),
  map((a) => a.age),
  reduce(add),
  console.log
);
```

- `go`로 순서를 반대로 뒤집고, `curry`로 보다 표현력 있고 간결하게 표현

<br />

## 함수 조합으로 함수 만들기

- 같은 데이터를 가지고 두 가지 일을 하는 코드들이 있다.
- 현재 중복이 있다. pipe 라인로 만들어진 코드를 쉽게 조합해서 중복을 제거할 수 있다.

```javascript
const go = (...args) => reduce((a, f) => f(a), args);

const pipe =
  (f, ...fs) =>
  (...as) =>
    go(f(...as), ...fs);

const total_age = pipe(
  map((a) => a.age),
  reduce(add)
);

go(
  attendance,
  filter((a) => a.age < 25),
  total_age,
  console.log
);

go(
  attendance,
  filter((a) => a.age >= 25),
  total_age,
  console.log
);
```

### 코드 더 쪼개보기

- 중복을 계속해서 제거해서 더 많은 곳에 쓰이게 할 수 있다.

```javascript
const go = (...args) => reduce((a, f) => f(a), args);

const pipe =
  (f, ...fs) =>
  (...as) =>
    go(f(...as), ...fs);

const total_age = pipe(
  map((a) => a.age),
  reduce(add)
);

const base_total_age = (predi) => pipe(filter(predi), total_age);

go(
  attendance,
  base_total_age((a) => a.age < 25),
  console.log
);

go(
  attendance,
  base_total_age((a) => a.age >= 25),
  console.log
);
```
