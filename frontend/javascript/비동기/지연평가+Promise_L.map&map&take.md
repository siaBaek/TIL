> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 지연 평가 + Promise - L.map, map, take

- 기본적으로 동기적 상황에서만 잘 작동하는 상태
- 비동기 상황에서도 잘 제어할 수 있도록 코드를 만들어보자

```javascript
go(
  [1, 2, 3],
  L.map((a) => a + 10),
  take(2),
  console.log
); // [11, 12]

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map((a) => a + 10),
  take(2),
  console.log
); // ["[object Promise]10", "[object Promise]10"] (error)
```

- 첫 번째 들어오는 인자가 프로미스라면 에러가 발생한다.
- 위 코드가 정상 작동하도록 해보자.

```javascript
// [기존 L.map] 이터러블 값을 순회하며 a에 f함수를 적용
L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield f(a);
  }
});

// [변경된 L.map] go1함수를 이용하여 a가 프로미스면 풀어서 f를 적용하도록 변경
L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield go1(a, f);
  }
});

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map((a) => a + 10),
  take(2),
  console.log
); // [Promise {<resolved>: 11}, Promise {<resolved>: 12}]
```

- 프로미스로서, 값이 아니라 값이 될 예정인 상태가 된다.
- take 함수에서 프로미스로 들어있는 값들을 값을 꺼내서 배열에 담을 수 있도록 해주자.

```javascript
// [기존 take] 이터러블을 받아서 이터레이터를 만들고, 순회하면서 안에 있는 값을 꺼내서 해당하는 값을 리턴할 res에 push하고, 리턴할 값의 length가 l과 비교해서 같아졌다면 즉시 리턴, 같아지지 않았다면 최대한 배열에 담은 다음 리턴을 하는 형태
const take = curry((l, iter) => {
  let res = [];
  iter = iter[Symbol.iterator]();
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(a); // 문제: a가 그대로 Promise
    if (res.length == l) return res;
  }
  return res;
});

// [변경된 take] a가 프로미스의 인스턴스인지 확인해주고 바로 리턴해주며 then을 이용하여 결과를 꺼내 push하고 개수가 같은지까지도 확인해준다. 최종 리턴이 되지 않았을 경우 다시 while문으로 돌아가야하는데, 이미 프로미스 인스턴스인지 확인하는 부분에서 리턴을 해서 함수를 빠져나갈 상황이 되어 버렸다.
// reduce 작성했을 때처럼 재귀할 부분을 잘라내고 return 해주고 유명함수로 감싸서 즉시 실행해주면 된다. 이후 while문으로 돌아갈 부분에서 다시 함수를 실행시키면 된다
const take = curry((l, iter) => {
  let res = [];
  iter = iter[Symbol.iterator]();
  return (function recur() {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      if (a instanceof Promise)
        return a.then((a) => ((res.push(a), res).length == l ? res : recur()));
      res.push(a);
      if (res.length == l) return res;
    }
    return res;
  })();
});

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map((a) => a + 10),
  take(2),
  console.log
); // [11, 12]
```

- 이제 L.map에 들어올 값이 [1,2,3]처럼 일반적인 값이었을 때도 동기적으로 동작이 잘 되고, 프로미스여도 동작이 잘 된다.
- 그런데 만약 L.map 내부 함수가 프로미스를 리턴했을 때는 어떻게 될까?

```javascript
// go1함수와 변경된 L.map 내부를 다시 봐보자
const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));

// a가 프로미스가 아닌 상태로 들어가서 go1이 실행된다
// go1는 리턴값으로 프로미스로 평가가 된다
// 결국 take 입장에서는 동일한 로직으로 동작하여 여전히 잘 동작하게 된다

L.map = curry(function* (f, iter) {
  for (const a of iter) {
    yield go1(a, f);
  }
});

go(
  [1, 2, 3],
  L.map((a) => Promise.resolve(a + 10)),
  take(2),
  console.log
); // [11, 12]
```

- 따라서 아래 코드 모두 정상 작동하게 된다.

```javascript
// L.map
go(
  [1, 2, 3],
  L.map((a) => Promise.resolve(a + 10)),
  take(2),
  console.log
); // [11, 12]

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map((a) => a + 10),
  take(2),
  console.log
); // [11, 12]

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  L.map((a) => Promise.resolve(a + 10)),
  take(2),
  console.log
); // [11, 12]

// map
go(
  [1, 2, 3],
  map((a) => Promise.resolve(a + 10)),
  console.log
); // [11, 12, 13]

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  map((a) => a + 10),
  console.log
); // [11, 12, 13]

go(
  [Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
  map((a) => Promise.resolve(a + 10)),
  console.log
); // [11, 12, 13]
```
