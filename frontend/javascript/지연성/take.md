> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## take

- iterable을 순회하며 0부터 l-1까지 총 l개를 배열에 담아주는 함수

```javascript
const range = (l) => {
  let i = -1;
  let res = [];
  while (++i < l) {
    res.push(i);
  }
  return res;
};

const L = {};
L.range = function* (l) {
  let i = -1;
  while (++i < l) {
    yield i;
  }
};

const take = (l, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == l) return res;
  }
  return res;
};
console.log(take(5, range(100))); // (5) [0, 1, 2, 3, 4]
console.log(take(5, L.range(100))); // (5) [0, 1, 2, 3, 4]
```

- L.range() 같이 지연성을 가지는 값을 이터레이터로 만들게 되면, 전혀 다른 함수(take)가 이터러블 프로토콜을 따른다면 조합이 가능하다.

<br />
<br />

## 효율성적인 측면

### take(5, range(100000)))

- 10000 크기의 배열을 만들고 5개를 뽑기 때문에 훨씬 비효율적이다.

### take(5, L.range(100000)))

- 최대 10000 크기의 배열을 뽑을 수 있는데, 5개만 뽑기 때문에 훨씬 효율적이다.
- 그렇기 때문에, L.range로 무한수열을 표현해도 어차피 정해진 양만 뽑기 때문에 가능하다.

### 성능 테스트 (전체코드)

```javascript
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

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

const go = (...args) => reduce((a, f) => f(a), args);

const pipe =
  (f, ...fs) =>
  (...as) =>
    go(f(...as), ...fs);

const add = (a, b) => a + b;

const range = (l) => {
  let i = -1;
  let res = [];
  while (++i < l) {
    res.push(i);
  }
  return res;
};

const L = {};
L.range = function* (l) {
  let i = -1;
  while (++i < l) {
    yield i;
  }
};

const take = curry((l, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == l) return res;
  }
  return res;
});

console.time('');
go(range(10000), take(5), reduce(add), console.log); // 10
console.timeEnd(''); // : 1.263916015625 ms

console.time('');
go(L.range(10000), take(5), reduce(add), console.log); // 10
console.timeEnd(''); // : 0.092041015625 ms
```

- 이런 지연성은 take나 reduce같은 함수를 만날 때 연산이 시작된다.
- 이터레이터를 리턴하는 함수를 실행했을 때는 해당 연산이 안에서 이루어지지 않고, take, reduce 등의 연산이 필요할 때 까지 최대한 연산을 미루다가 만났을 때 연산을 처음하게 된다.
