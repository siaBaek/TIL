> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 즉시, 지연, Promise, 병렬적 조합하기

- 지금까지 즉시 평가 되는 이터러블 중심 함수들과 지연 평가되는 함수들, 지연+즉시에서 Promise 지원하는 코드, 병렬적으로 실행하고 다루는 코드들 작성해보았다.
- 조합해서 원하는 평가 전략을 세워보자.

### 동기 + 엄격한 평가(즉시 평가)

```javascript
const delay500 = (a) => a;

// take
console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map((a) => delay500(a * a)),
  filter((a) => delay500(a % 2)),
  map((a) => delay500(a + 1)),
  take(2),
  console.log,
  (_) => console.timeEnd('')
);
// [2, 10]
// 4.9919ms

// reduce
console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map((a) => delay500(a * a)),
  filter((a) => delay500(a % 2)),
  map((a) => delay500(a + 1)),
  reduce(add),
  console.log,
  (_) => console.timeEnd('')
);
// 88
// 2.9079ms
```

<br />

### 비동기(Promise) + 엄격한 평가(즉시 평가)

```javascript
const delay500 = (a, name) =>
  new Promise((resolve) => {
    console.log(`${name}: ${a}`);
    setTimeout(() => resolve(a), 500);
  });

// take
console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map((a) => delay500(a * a, 'map 1')),
  filter((a) => delay500(a % 2, 'filter 2')),
  map((a) => delay500(a + 1, 'map 3')),
  take(2),
  console.log,
  (_) => console.timeEnd('')
);
// (500ms 간격)
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// map 3: 2
// map 3: 10
// map 3: 26
// map 3: 50
// [2, 10]
// 10064.8940ms

// reduce
console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  map((a) => delay500(a * a, 'map 1')),
  filter((a) => delay500(a % 2, 'filter 2')),
  map((a) => delay500(a + 1, 'map 3')),
  reduce(add),
  console.log,
  (_) => console.timeEnd('')
);
// (500ms 간격)
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// map 3: 2
// map 3: 10
// map 3: 26
// map 3: 50
// 88
// 10062.9221ms
```

<br />

### 비동기(Promise) + 지연 평가

```javascript
const delay500 = (a, name) =>
  new Promise((resolve) => {
    console.log(`${name}: ${a}`);
    setTimeout(() => resolve(a), 500);
  });

console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => delay500(a * a, 'map 1')),
  L.filter((a) => delay500(a % 2, 'filter 2')),
  L.map((a) => delay500(a + 1, 'map 3')),
  take(2),
  console.log,
  (_) => console.timeEnd('')
);
// (500ms 간격)
// map 1: 1
// filter 2: 1
// map 3: 2
// map 1: 4
// filter 2: 0
// map 1: 9
// filter 2: 1
// map 3: 10
// [2, 10]
// 4026.9196ms
```

<br />

### 비동기(Promise) + 지연 평가 + 병렬 평가

```javascript
const delay500 = (a, name) =>
  new Promise((resolve) => {
    console.log(`${name}: ${a}`);
    setTimeout(() => resolve(a), 500);
  });

console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  C.map((a) => delay500(a * a, 'map 1')),
  L.filter((a) => delay500(a % 2, 'filter 2')),
  L.map((a) => delay500(a + 1, 'map 3')),
  take(3),
  console.log,
  (_) => console.timeEnd('')
);
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// (500ms 간격)
// filter 2: 1
// map 3: 2
// filter 2: 0
// filter 2: 1
// map 3: 10
// filter 2: 0
// filter 2: 1
// map 3: 26
// [2, 10, 26]
// 4542.1809ms

console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => delay500(a * a, 'map 1')),
  C.filter((a) => delay500(a % 2, 'filter 2')),
  L.map((a) => delay500(a + 1, 'map 3')),
  take(3),
  console.log,
  (_) => console.timeEnd('')
);
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// (500ms 간격)
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// map 3: 2
// map 3: 10
// map 3: 26
// [2, 10, 26]
// 2521.0178ms

// take
console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => delay500(a * a, 'map 1')),
  L..filter((a) => delay500(a % 2, 'filter 2')),
  L.map((a) => delay500(a + 1, 'map 3')),
  C.take(3),
  console.log,
  (_) => console.timeEnd('')
);
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// (500ms 간격)
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// (500ms 간격)
// map 3: 2
// map 3: 10
// map 3: 26
// [2, 10, 26]
// 1511.4431ms

// reduce

console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => delay500(a * a, 'map 1')),
  L..filter((a) => delay500(a % 2, 'filter 2')),
  L.map((a) => delay500(a + 1, 'map 3')),
  C.reduce(3),
  console.log,
  (_) => console.timeEnd('')
);
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// (500ms 간격)
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// (500ms 간격)
// map 3: 2
// map 3: 10
// map 3: 26
// map 3: 50
// 88
// 1510.4111ms

```

<br />

### 비동기(Promise) + 엄격한 평가(즉시 평가) + 지연 평가 + 병렬 평가

```javascript
const delay500 = (a, name) =>
  new Promise((resolve) => {
    console.log(`${name}: ${a}`);
    setTimeout(() => resolve(a), 500);
  });

console.tiem('');
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => delay500(a * a, 'map 1')),
  C.filter((a) => delay500(a % 2, 'filter 2')),
  map((a) => delay500(a + 1, 'map 3')),
  take(2),
  console.log,
  (_) => console.timeEnd('')
);
// map 1: 1
// map 1: 4
// map 1: 9
// map 1: 16
// map 1: 25
// map 1: 36
// map 1: 49
// map 1: 64
// (500ms 간격)
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// filter 2: 1
// filter 2: 0
// (500ms 간격)
// map 3: 2
// (500ms 간격)
// map 3: 10
// (500ms 간격)
// map 3: 26
// (500ms 간격)
// map 3: 50
// (500ms 간격)
// [2, 10]
// 2015.9621ms
```

- Node.js에서 이미지 연산하거나 DB 다녀오거나 등을 함수와 표현식으로 이터러블하게 해당 코드를 값으로 다루며 병렬적으로 평가할 것인지, 지연적으로 평가할 것인지, 부하를 많이주고 빨리 결과를 얻을 것인지, 평가 자체를 최소화할 것인지, 모두 엄격하게 평가할 것인지... 등을 코드로 다룰 수 있게 되었다.
