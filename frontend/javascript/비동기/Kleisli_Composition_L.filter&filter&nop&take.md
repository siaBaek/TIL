> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## Kleisli Composition - L.filter, filter, nop, take

- filter에서 지연평가와 프로미스를 함께 지원하려면, Kleisli composition을 활용해야한다.
- L.filter에 주어진 값이 프로미스가거나, L.filter가 리턴하는 값이 프로미스일 때, 어떻게 지연 평가도 되면서 비동기 상황의 프로미스도 모두 지원할 수 있을까?

```javascript
// 주어진 값이 프로미스
go(
  [1, 2, 3, 4, 5, 6],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => {
    console.log(a); // Promise {<resolved>: 1,4,9,16,25,36}
    return a % 2;
  }),
  take(2),
  console.log
); // []

// [기존 L.filter]
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    // 여기서 들어가는 a 값이 프로미스이므로
    if (f(a)) yield a; // f에 전달될 때 프로미스를 해결한 채로 전달되어야 한다
  }
});

// [변경된 L.filter - 1]
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    // 여기서 a 값이 프로미스이므로
    const b = go1(a, f);
    if (f(b)) yield a; // f에 전달될 때 프로미스를 해결한 채로 전달되어야 한다
  }
});

// 위 코드처럼 L.filter를 변경하면
// 동기적 상황에서 잘 동작하게 된다
go(
  [1, 2, 3, 4, 5, 6],
  //   L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => {
    console.log(a); // 1, 2, 3
    return a % 2;
  }),
  take(2),
  console.log
); // [1, 3]

// 비동기 상황에서는 정상 동작하지 않는다
go(
  [1, 2, 3, 4, 5, 6],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => {
    console.log(a); // 1, 4
    return a % 2;
  }),
  take(2),
  console.log
);
```

- a를 go1 함수를 한 번 거친 b를 f에 전달하면 동기적 상황에서는 잘 동작하게 되지만, 비동기 상황에서는 잘 동작하지 않게 된다.
- 이 때, b가 프로미스일 때를 구분하여 작성해주면 해결할 수 있다.

```javascript
// [변경된 L.filter - 2]
const nop = Symbol('nop');
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    const b = go1(a, f);
    // b가 true면 a(프로미스) 전달: 프로미스인 채로 보내도 다른 곳에서 then으로 풀어서 쓸 것이기 때문에 a만 전달해주면 된다
    // b가 false면 Promise.reject()을 해주어 이후 함수가 실행될 일이 없게 된다
    // 아무 일도 하지 않길 바라는 reject인지 완전히 에러 reject인지 구분하기 위해, 이후에 어떤 일을 계속 하고 싶으면 정상 값을 전달하고 그게 아니면 reject에 특정 값을 담아서 전달해주면 된다 (nop이라는 구분자를 이용)
    // 이후 일어날 함수 대기열을 아무 것도 실행하지 않고 끝내는 방법 -> Kleisli Composition

    if (b instanceof Promise) yield b.then((b) => (b ? a : Promise.reject(nop)));
    // catch해서 꺼냈을 때 nop이 오면 아무 일도 하지 않도록 처리해주면 된다
    else if (f(b)) yield a;
  }
});

// take 함수에서 catch를 통해 꺼낼 수 있다.
const take = curry((l, iter) => {
  let res = [];
  iter = iter[Symbol.iterator]();
  return (function recur() {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      if (a instanceof Promise) {
        return a
          .then((a) => ((res.push(a), res).length == l ? res : recur()))
          .catch((e) => (e == nop ? recur() : Promise.reject(e)));
        // 에러로 전달된 값이 nop이면 아무 일도 안하고 다음 코드 평가
        // 에러로 전달된 값이 nop이 아니라 실제로 에러가 났다면 다시 그 에러로 전달
      }
      res.push(a);
      if (res.length == l) return res;
    }
    return res;
  })();
});

go(
  [1, 2, 3, 4, 5, 6],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => {
    console.log(a); // 1, 4
    return a % 2;
  }),
  take(2),
  console.log
);
```

- catch를 하면 아무리 then, then, then을 달아도 해당 함수 이후에는 함수 전달이 되지 않는다.

```javascript
Promise.resolve(1)
  .then(() => Promise.reject('err')) // reject한 경우 이후 일어나는 모든 함수 대기열 전혀 실행하지 않고
  .then(() => console.log('여기'))
  .then(() => console.log('여기'))
  .then(() => console.log('여기'))
  .catch((e) => console.log(e, 'hi')); // catch가 실행되게 된다
// err hi
```

- 따라서, L.filter에서 reject(nop)한 라인(L.filter에서 false가 된 경우)은 아래 있는 함수 대기열이 실행되지 않고 take로 가게 된다.
- 이후 take 내부에서 catch문에 의해 recur()가 실행되어 next()가 되면서 계속해서 연산이 실행되게 된다.

```javascript
go(
  [1, 2, 3, 4, 5, 6],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => a % 2),
  L.map((a) => a * a),
  take(2),
  console.log
); // [1, 81]

go(
  [1, 2, 3, 4, 5, 6],
  L.map((a) => Promise.resolve(a * a)),
  filter((a) => Promise.resolve(a % 2)),
  console.log
); // [1, 9, 25]
```

- 이제 filter함수는 지연 평가, 동기, 비동기 상황 모두 처리할 수 있게 되었다.
