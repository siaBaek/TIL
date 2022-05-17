> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 지연된 함수열을 병렬적으로 평가하기 - C.reduce, C.take

- 자바스크립트 동작 환경에선 [비동기 I/O](https://ko.wikipedia.org/wiki/%EB%B9%84%EB%8F%99%EA%B8%B0_%EC%9E%85%EC%B6%9C%EB%A0%A5)로 동작하는데, 이는 싱글 스레드 기반으로 I/O들을 동기적으로 처리하기 보다는 비동기적으로 처리해서 하나의 스레드에서도 CPU 등의 시스템 자원을 더욱 효율적으로 사용하게 하는 것을 의미한다.
- 자바스크립트는 싱글 스레드 기반이기 때문에 병렬적 프로그래밍을 할 일이 없다고 생각할 수 있지만, 사실 자바스크립트가 어떤 로직을 제어하는 것을 싱글 스레드로 비동기적으로 제어할 뿐이지 얼마든지 병렬적인 처리가 필요할 수 있다.
- 예를 들어, Node.js에서 [postgreSQL](https://ko.wikipedia.org/wiki/PostgreSQL)같은 DB에 날리는 쿼리를 병렬적으로 출발시켜 결과를 한 번에 얻어온다거나, [Redis](https://ko.wikipedia.org/wiki/%EB%A0%88%EB%94%94%EC%8A%A4) 같은noSQL DB에서 여러 key들을 가지고 여러 결과를 한 번에 처리를 한다거나, 이미지 처리를 Node.js에서 처리할 경우는 실제 Node.js가 직접 처리하는 것이 아니라 네트워크나 기타I/O로 작업을 보내놓고 대기하고 그 시점을 다루는 일만 Node.js가 하게 된다.
- 이처럼 어떤 처리들을 동시에 출발시켜 하나의 로직으로 귀결시키는 등의 로직은 개발자가 자바스크립트에서 잘 다룰 필요가 있다. (물론 동시성으로 동작하는 병렬성을 의미한다)

```javascript
const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a), 1000);
  });

// 아래 코드로 실행한다면
// map을 통해 배열 전체를 500*5ms가 걸려 평가한 배열을 필터해서 모두 평가하고 reduce에서 하나씩 더해서 평가하게 될 것이다
go(
  [1, 2, 3, 4, 5],
  map((a) => delay1000(a * a)),
  filter((a) => a % 2),
  reduce(add),
  console.log
); // 35

// 아래 코드를 실행한다면,
// 1->map(1)->filter(true)->add, 2->map(4)->filter(false) 처럼 세로로 평가가 될 것 이다
// 비동기 상황이 발생시, reduce에서 대기 중인 함수는 reduce에서 첫 번째 값을 꺼낼 때 평가가 이루어 진다는 것
go(
  [1, 2, 3, 4, 5],
  L.map((a) => delay500(a * a)), // 함수 실행 전 대기 중인 상태
  L.filter((a) => a % 2), // 함수 실행 전 대기 중안 상태
  reduce(add),
  console.log
); // 35
```

<br />

### C.reduce

- 지금은 하나 씩 값을 기다려서 다음, 다음으로 가고 있는데 모두 동시에 출발시킨 다음에 더하는 reduce를 만든다면 훨씬 효율적일 것이다.
- 함수형 프로그래밍에서는 상태나 외부 값에 의존하지 않기 때문에 병렬코드 로직이 단순하다.

```javascript
// C = Concurrency(병행성)
const C = {};
// f:함수 acc:누적할 값 iter:순회할 이터러블(option)
// iter가 없다는 것은 acc가 iter라는 것
C.reduce = curry((f, acc, iter) => {
  return iter
    ? reduce(f, acc, [...iter]) // 대기된 함수 비동기 제어하지 않고 모두 실행하고 다시 reduce 순회
    : reduce(f, [...acc]);
});

const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a), 1000);
  });

go(
  [1, 2, 3, 4, 5],
  L.map((a) => delay1000(a * a)),
  L.filter((a) => a % 2),
  C.reduce(add),
  console.log
);
// <5> hi - 5번 동시 실행
// 35
```

#### reduce, C.reduce의 연산 속도 차이

- reduce 사용시 연산 속도: 5013.5771...ms
- C.reduce 사용시 연산 속도: 1005.9492...ms
- delay1000이라는 작업이 Node.js에서 일어나는 것이 아니라 외부 환경에서 병렬적으로 처리해도 되는 상황이고, 해당 작업이 부하가 아니라 효율인 상황이라면 얼마든지 병렬 평가가 훨씬 효율성을 가질 수 있다.

<br />

### L.filter 에서도 연속적으로 딜레이가 있고, 이후에도 함수 대기열이 있었다면?

```javascript
const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a), 1000);
  });

go(
  [1, 2, 3, 4, 5, 6, 7, 8, 9],
  L.map((a) => delay1000(a * a)),
  L.filter((a) => delay1000(a % 2)),
  L.map((a) => delay1000(a * a)),
  C.reduce(add),
  console.log
);
// <9> hi
// <9> hi
// <2> hi
// Uncaught (in promise) Symbol(nop)
// hi
// Uncaught (in promise) Symbol(nop)
// hi
// Uncaught (in promise) Symbol(nop)
// hi
// 9669
```

#### 자바스크립트 프로미스의 특성

- 연산되는 값 자체는 문제 없이 동작하지만 catch 되지 않은 부분이 생긴다
- Promise.reject('hi!')를 실행시 `Uncaught (in promise) hi!` 발생
- 콜스택에 프로미스 reject으로 평가되는 코드가 있으면 출력하게 되어 있다.
- 해당 값을 나중에 catch를 해도 이미 찍혀서 코드가 출력된다.
- 나중에 해결을 한다고 하더라도 이미 출력된 로그는 어떻게 할 수가 없다.

```javascript
// 해결될 것이기 때문에 프로미스에서 콜스택에 에러 찍을 필요 없다고 설명해줘야 한다
// 그 부분은 다음 콜스택에서 처리할거야! 비동기적으로 해당 에러를 캐치할거야!
C.reduce = curry((f, acc, iter) => {
  return iter
    ? reduce(f, acc, [...iter]) // 여기서 Promise reject으로 평가될 수 있는 상황
    : // reject 일어나기 전에 미리 catch 해줘야 한다
      // reject가 일어난 후에 catch하면 에러 캐치는 되지만 필요 없는 로그가 찍히는 것이다
      reduce(f, [...acc]);
});

// 미리 catch를 해보자
C.reduce = curry((f, acc, iter) => {
  const iter2 = iter ? [...iter] : [...acc];
  iter2.forEach((a) => a.catch(function () {})); // 에러가 나도 딱히 아무 일도 안할거야!
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});

const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a), 1000);
  });

go(
  [1, 2, 3, 4, 5, 6, 7, 8, 9],
  L.map((a) => delay1000(a * a)),
  L.filter((a) => delay1000(a % 2)),
  L.map((a) => delay1000(a * a)),
  C.reduce(add),
  console.log
);
// <9> hi
// <9> hi
// <5> hi
// 9669
```

- 중요한 것은, catch가 된 프로미스들을 전달한다면 이후에 다시 캐치를 할 수가 없어서 처리할 수 없다.
- catch를 한 프로미스를 전달하는 것이 아니라 cath를 하지 않은 프로미스를 전달하되, 임시적으로 앞에서 미리 catch를 해두기만 하는 것이다.

```javascript
C.reduce = curry((f, acc, iter) => {
  var iter2 = iter ? [...iter] : [...acc];
  iter2 = iter2.map((a) => a.catch(function () {}));
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});

const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a), 1000);
  });

go(
  [1, 2, 3, 4, 5, 6, 7, 8, 9],
  L.map((a) => delay1000(a * a)),
  L.filter((a) => delay1000(a % 2)),
  L.map((a) => delay1000(a * a)),
  C.reduce(add),
  console.log
);
// <9> hi
// <9> hi
// <5> hi
// NaN
```

```javascript
var a = Promise.reject('hi');
a = a.catch((a) => a);

// 위 코드는 아래 코드와 같고

var a = Promise.reject('hi').catch((a) => a);
// 즉시 a값 자체가 catch까지 한 프로미스가 된다
// 이렇게 되면, 이후에 catch를 또 할 수가 없게 된다

// 아래 코드처럼 catch만 걸어놓고 에러가 나지 않도록만 해두면
// a는 reject까지만 되어있고 아직 catch를 하지 않은 값이기 때문에 이후에 원하는 시점에 catch를 할 수 있게 된다
var a = Promise.reject('hi');
a.catch((a) => a);
a.catch((a) => console.log(a, 'c')); // hi c
```

#### 코드정리

- noop, catchNoop 함수를 이용하여 정리할 수 있다.

```javascript
function noop() {}
const catchNoop = (arr) => (arr.forEach((a) => (a instanceof Promise ? a.catch(noop) : a)), arr);

C.reduce = curry((f, acc, iter) => {
  var iter2 = catchNoop(iter ? [...iter] : [...acc]);
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});

// 이렇게 표현하는 것보다 함수를 연속적으로 실행하는 식으로 표현하는 게 대부분 좋다
// if를 두번 나누는 것 보다 아래와 같이 합쳐주는 것이 보기에도 깔끔하다

C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, catchNoop([...iter])) : reduce(f, catchNoop([...iter]))
);

// 만약 catchNoop을 C.reduce 안에서만 사용한다고 가정한다면,
C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, catchNoop(iter)) : reduce(f, catchNoop(iter))
);
C.take = curry((l, iter) => take(l, catchNoop(iter)));
// 이렇게 iter로 보내주고, catchNoop을 수정해주어도 된다
const catchNoop = ([...arr]) => (
  arr.forEach((a) => (a instanceof Promise ? a.catch(noop) : a)), arr
);

// [참고] 전개 연산자를 사용한다면 아래 코드처럼 작성할 수도 있다
C.reduce = curry((f, acc, iter) =>
  reduce(f, ...(iter ? [acc, catchNoop(iter)] : [catchNoop(acc)]))
);
```

- 이제 함수 대기열에 상관 없이 병렬적으로 실행하게 된다.
- 원래 세로로 평가하던 것을 최대한 병렬적으로 실행하게 해서 싱글스레드 기반이지만 외부에게 명령 전달할 때 제어를 통해 코드 효율성을 높일 수 있다.

<br />

### C.take

- 같은 방식으로 작성해보자.

```javascript
C.take = curry((l, iter) => {
  iter = catchNoop([...iter]);
  return take(l, iter);
});

C.take = curry((l, iter) => take(l, catchNoop([...iter])));

// 이제 받은 값을 가지고 모두 즉시 평가 하되, 안전하게 알아서 catch할테니 에러 띄우지 말라고 할 수 있다


go(
  [1, 2, 3, 4, 5, 6, 7, 8, 9],
  L.map((a) => delay1000(a * a)),
  L.filter((a) => delay1000(a % 2)),
  L.map((a) => delay1000(a * a)),
  C.take(2)
  C.reduce(add),
  console.log
);
// <9> hi
// <9> hi
// <5> hi 병렬적으로 모두 실행을 하고
// 82 두개의 값만 꺼내서 처리
```

#### take, reduce와 C.take, C.reduce차이

- take, reduce
  - 부하를 줄이고, 명령 자체를 덜 실행시키는 방향으로 최적화한다.
- C.take, C.reduce
  - 최대한 많은 자원을 써서 최대한 빠르게 결과를 꺼낸다.
