> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## go, pipe, reduce에서 비동기 제어

- 동기 상황과 비동기 상황 모두 잘 대응하는 [go1 함수](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/%EB%B9%84%EB%8F%99%EA%B8%B0/callback%EA%B3%BC_Promise%EC%9D%98_%EC%B0%A8%EC%9D%B4.md)
- [Kleisli composition](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/%EB%B9%84%EB%8F%99%EA%B8%B0/callback%EA%B3%BC_Promise%EC%9D%98_%EC%B0%A8%EC%9D%B4.md)
- go, pipe, reduce -> 어떤 함수를 연속적으로 실행하는 함수 합성에 대한 함수
- Promise의 비동기를 일급값으로 다루는 성질을 이용해서, 비동기 상황에 놓여져도 잘 대응하는 함수를 만든다거나 Kleisli composition처럼 중간에 reject가 일어나거나 에러났을 때 대기중인 이후 함수들을 실행하지 않고 맨 뒤로 보내는 기법을 적용할 수 있다.

```javascript
go(
  1,
  (a) => a + 10,
  (a) => Promise.resolve(a + 100), // 비동기 상황. Promise 리턴
  (a) => a + 1000,
  console.log
); // [object Promise]1000 (비정상 동작)
```

- 위 코드를 정상 동작하게 하기 위해 go1 만들 때 사용했던 기법을 사용할 수 있다.
- go는 reduce를 사용하고 즉시 함수 실행 외에는 다른 문장이 없다.
- 모든 제어권은 reduce가 가지고 있고 reduce 안에서는 어떠한 값 확인을 하지 않기 때문에, reduce만 고쳐주면 정상 동작하도록 할 수 있다.

```javascript
// for (const a of iter) 부분을 풀어준 reduce
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  // 내부 값을 done이 아닐 때까지 계속 순회하며 받아둔 함수를 실행하고 실행된 값은 acc가 되는 형태
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    // acc = f(acc, a);
    // go함수 실행시 acc가 어느 시점에는 Promise가 된다
    // 함수 첫 번째 인자에 프로미스를 전달한채로 함수를 실행하기 때문에 프로미스 값을 기다려서 만들어진 값으로 변환해주어야 연속적으로 실행이 가능해진다
    acc = acc instanceof Promise ? acc.then((acc) => f(acc, a)) : f(acc, a);
  }
  return acc;
});

go(
  1,
  (a) => a + 10,
  (a) => Promise.resolve(a + 100), // 비동기 상황. Promise 리턴
  (a) => a + 1000,
  (a) => a + 10000,
  console.log
); // 11111
```

- 위 코드는 중간에 프로미스를 한 번 만나게 되면, 이후 함수에서도 프로미스 체인에 함수를 합성하기 때문에 연속적으로 비동기가 일어나게 된다.
- 개발자가 이후 함수의 경우 동기적으로 즉시 하나의 콜스택에서 실행되기를 바랐다면, 위 코드로는 처리할 수 없다.
- 이러한 함수 합성이 코드에 많다면 불필요한 로드가 생겨 성능 저하가 있게 된다.
- 중간에 프로미스가 만나도, 프로미스가 아닐 경우 동기적으로 계속해서 while문을 통해 다음으로 넘어갈 수 있도록 구성해줘야 한다.

```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  // 재귀를 이용. 유명함수 리턴하여 즉시 실행
  // 유명함수: 함수를 값으로 다루면서 함수에 이름을 짓는 기법
  return (function reucr(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur); // 해당 acc로 변경되어 recur 실행
    }
    return acc;
  })(acc);
});

go(
  1,
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  console.log
); // 11111

go(
  Promise.resolve(1),
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  console.log
); // [object Promise]10100100010000
```

- 위 코드는 처음으로 들어갈 acc가 프로미스일 경우 에러가 발생한다.
- 처음 만들어낸 acc 자체가 이미 프로미스이기 때문에 에러가 발생하는 것이다.
- 이 부분을 해결하려면 recur가 처음 실행될 때 프로미스가 풀려서 실행되어야 할 것이다.
- go1 함수를 이용할 수 있다.

```javascript
const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  // 처음 acc가 프로미스면 풀어서 전달하게 되고,
  // 아니라면 go1에서 하나의 콜스택으로 이어지도록 동기적으로 실행하기 때문에 더 효율적
  return go1(acc, function reucr(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur); // 해당 acc로 변경되어 recur 실행
    }
    return acc;
  });
});

go(
  Promise.resolve(1),
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  console.log
); // 11111

go(
  Promise.resolve(1),
  (a) => a + 10,
  (a) => Promise.reject('error!'),
  (a) => a + 1000, // 실행안됨
  (a) => a + 10000, // 실행안됨
  console.log
); // Uncaught (in promise) error!

go(
  Promise.resolve(1),
  (a) => a + 10,
  (a) => Promise.reject('error!'),
  (a) => a + 1000, // 실행안됨
  (a) => a + 10000, // 실행안됨
  console.log
).cath((a) => console.log(a)); // error!
```

- 단순히 프로미스를 콜백지옥을 해결하는 용도로만 사용하는 것이 아닌, 프로미스라는 값을 가지고 원하는 로직을 사용한다거나, 원하는 시점에서 원하는 방식으로 받아둔 함수를 실행시키는 고차함수를 만들 수 있다.
