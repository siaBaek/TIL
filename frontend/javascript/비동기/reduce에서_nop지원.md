> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## reduce에서 nop 지원

- reduce에서도 take처럼 이터러블 중심 프로그래밍에서 용이하게 값을 다룰 수 있도록 nop을 지원해서 지연성과 프로미스를 모두 지원하도록 해보자.

```javascript
go(
  [1, 2, 3, 4],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => Promise.resolve(a % 2)),
  reduce(add),
  console.log
); // 1[object Promise][object Promise][objectPromise]

// [기존 reduce]
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  return go1(acc, function reucr(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a); // a가 프로미스인 상태로 전달되어 add함수 실행 시 에러
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});

// [변경된 reduce]
const reduceF = (acc, a, f) => {
  // acc는 프로미스가 아닌 상태로 받기 때문에 a만 체크해서 풀어주면 된다
  // then할 때 두 번째 인자는 reject를 의미하게 된다
  a instanceof Promise
    ? a.then(
        (a) => f(acc, a),
        (e) => (e == nop ? acc : Promise.reject(e))
      )
    : f(acc, a);
};

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  return go1(acc, function reucr(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      //   const a = cur.value;
      //   acc = f(acc, a);
      acc = reduceF(acc, cur.value, f);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});

// 이제 reduce에 전달된 값이 프로미스여도 잘 동작하게 된다
go(
  [1, 2, 3, 4, 5],
  L.map((a) => Promise.resolve(a * a)),
  L.filter((a) => Promise.resolve(a % 2)),
  reduce(add),
  console.log
); // 35
```

- 이제 다양한 경우의 수를 잘 합성해주는 함수가 될 수 있도록 문장이 아닌 표현식으로 정리해주자.
- next() 꺼낼 때도 비동기가 일어난다면 안전하게 합성하기 위해 프로미스를 잘 다룰 수 있도록 해주어야 한다.
- 표현식으로 코딩하는 이유는 최대한 다형성을 지원하고 안전하게 에러를 흘려보내기 위해서이다.

```javascript
const reduceF = (acc, a, f) => {
  a instanceof Promise
    ? a.then(
        (a) => f(acc, a),
        (e) => (e == nop ? acc : Promise.reject(e))
      )
    : f(acc, a);
};

// 헤드 뽑는 함수 작성
// take를 하고 비동기 상황이 일어난다고 하더라도 구조 분해로 꺼내서 안쪽 값을 전달
const head = (iter) => go1(take(1, iter), ([h]) => h);

const reduce = curry((f, acc, iter) => {
  // 이터레이터를 만들고, next()를 통해 결과를 꺼내고 reduce 이후 일들 처리
  // 이터레이터에서 헤드를 뽑고, 헤드가 뽑혀진 나머지 이터레이터로 reduce
  if (!iter) return reduce(f, head((iter = acc[Symbol.iterator]()), iter));
  // head를 적용한 값을 reduce에게 전달해주면서 head가 acc가 되고, 거기까지 된 후의 이터레이터를 리턴하는 식으로 작성할 수 있다 (if에서 return하기 때문에 else문 삭제)
  iter = iter[Symbol.iterator]();
  return go1(acc, function reucr(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      acc = reduceF(acc, cur.value, f);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});
```
