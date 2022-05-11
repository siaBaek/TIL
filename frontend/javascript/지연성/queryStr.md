> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

- reduce, take가 실제로 지연성이나 map, filter와 어떻게 어우러져서 사용되는가

## reduce - queryStr, join

- queryStr 함수: 객체로부터 url의 query String 부분을 얻어내는 함수. reduce로 결론을 만들어내는 함수

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

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    acc = f(acc, a);
  }
  return acc;
});

const go = (...args) => reduce((a, f) => f(a), args);

const queryStr = (obj) =>
  go(
    obj,
    Object.entries,
    map(([k, v]) => `${k}=${v}`), // ['limit=10', 'offset=10', 'type=notice']
    reduce((a, b) => `${a}&${b}`) // limit=10&offset=10&type=notice
  );
console.log(queryStr({limit: 10, offset: 10, type: 'notice'}));
```

<br />

### go -> pipe 함수로 변경하기

```javascript
const queryStr = pipe(
  Object.entries,
  map(([k, v]) => `${k}=${v}`),
  reduce((a, b) => `${a}&${b}`)
);
```

- reduce 함수 같은 경우, Array의 join 함수를 떠올릴 수 있다.
- join 함수는 Array.prototype에만 붙어있고, reduce는 이터러블 값을 다 순회하며 축약할 수 있기 때문에 더 다형성이 높다.

<br />

### 다형성이 높은 join 함수

- reduce 계열의 join 함수를 만들어보자.
- Array.prototype.join() 은 배열에서만 사용할 수 있다.
- 배열이 아닌 것도 사용할 수 있는 join 함수를 만들어보자.

```javascript
const join = curry((sep = ',', iter) => reduce((a, b) => `${a}${sep}${b}`, iter));

const queryStr = pipe(
  Object.entries,
  map(([k, v]) => `${k}=${v}`),
  join('&')
);

function* a() {
  yield 10;
  yield 11;
  yield 12;
  yield 13;
}

a().join(','); // 불가능
console.log(join(','), a()); // 가능
```

- 받는 값을 reduce로 축약하기 때문에 배열이 아니어도 가능하다

<br />

- 이터러블 프로토콜을 따르고 있기 때문에 join에게 가기 전 값을 지연할 수 있다. (L.map 사용가능)
- 이유는, reduce에서 하나씩 next()를 통해 이터레이터의 값을 꺼낼 것이기 때문이다.

<br />

#### entries, map에 지연성 포함시키기

- join이 실행되어야 실행되는 함수가 되었다.
- 결과는 같지만 더 효율적이다.

```javascript
L.entries = function* (obj) {
  for (const k in obj) yield [k, obj[k]];
};

var it = L.entries({limit: 10, offset: 10, type: 'notice'});
it.next().value; // ["limit", 10]
it.next().value; // ["offset", 10]
it.next().value; // ["type", "notice]

const queryStr = pipe(
  L.entries,
  L.map(([k, v]) => `${k}=${v}`),
  join('&')
);
```

## take - find

- take를 통해 find 함수를 만들어보자.
- take로 결론을 만들어내는 함수

```javascript
const users = [
  {age: 32},
  {age: 31},
  {age: 37},
  {age: 28},
  {age: 25},
  {age: 32},
  {age: 31},
  {age: 37},
];

// find: 객체 배열로부터 특정 조건을 만족하는 해당 객체를 뽑아낸다
const curry =
  (f) =>
  (a, ..._) =>
    _.length ? f(a, ..._) : (..._) => f(a, ..._);

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

const take = curry((l, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == l) return res;
  }
  return res;
});

const find = (f, iter) =>
  go(
    iter,
    filter((a) => f),
    take(1),
    ([a]) => a
  );

// 첫 번째 하나의 값만 꺼내주는 함수
console.log(find((u) => u.age < 30, users));
// {age: 28}
```

- 아쉬운 점: 하나의 결과만 꺼내지만 모두 돌고 있다.
- L.filter로 변경하면 효율적으로 코드를 짤 수 있다.

#### filter에 지연성 포함시키기

- 결과는 같지만 더 효율적이다.

```javascript
const find = curry(f, iter) => go(iter, L.filter(f), take(1), ([a]) => a);

console.log(find((u) => u.age < 30)(users));

// 이렇게도 찾을 수 있다.
go(users,
    L.map((u)=>u.age),
    find(n=>n<30),
    console.log);
```
