> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## Array.prototype.map이 있는데 왜 FxJS의 map 함수가 필요한지?

- 아무리 async:await, Array.prototype.map이 있어도, 함수 콜에 있어서 비동기를 잘 제어하도록 고려되어 있는 라이브러리가 아니라면 async:await를 쉽게 사용할 수 없다.
- 따라서 Promise를 잘 다루는 식으로 만들어진 함수를 사용할수록 편하게 사용할 수 있다.

```javascript
async function delayI(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 100));
}

function f1() {
  const list = [1, 2, 3, 4];
  const res = list.map((a) => delayI(a * a));
  // res.reduce(add);
  // res가 Promise 배열로 평가되어 reduce(add)를 하려고 해도 정상 동작하지 않는다
  console.log(res);
}
f1(); // [Promise, Promise, Promise, Promise]
```

### 해결 방법

- 기존 map은 내부적으로 보조함수에서 Promise가 리턴되거나, 보조함수를 async 함수로 선언한다고 해서 map이 Promise를 제어해준다거나 하지 않는다.

```javascript
async function f2() {
  const list = [1, 2, 3, 4];
  const res = await list.map(async (a) => await delayI(a * a));
  console.log(res);
}
f2(); // [Promise, Promise, Promise, Promise]
```

- 따라서 위 코드처럼 아무리 async:await를 사용한다고 하더라도 안에서 비동기 상황을 제어해줄 수 없다
- 애초에 함수 자체가 비동기를 잘 제어하는 함수가 되어야 하는데, 아래와 같이 새로 만든 map을 이용하여 작성할 수 있다.

```javascript
async function f3() {
  const list = [1, 2, 3, 4];
  const res = await map((a) => delayI(a * a), list);
  console.log(res);
}
f3(); // [1, 4, 9, 16]
```

### Array.prototype.map은 왜 비동기 제어가 안되는지 알아보자

- f2()의 temp: Promise가 들어있는 Array를 주어서 이후 await으로 프로미스를 풀려고해도 풀 수가 없다.
- f3()의 temp: map을 했을 때 안쪽에 있는 값을 변경하고 그 값이 최종적으로 resolved 됐을 때 Array로 떨어질 준비가 된 map

```javascript
async function delayI(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 100));
}

async function f2() {
  const list = [1, 2, 3, 4];
  const temp = list.map(async (a) => await delayI(a * a));
  console.log(temp); // [Promise, Promise, Promise, Promise]
  const res = await temp;
  console.log(res);
}
f2();

async function f3() {
  const list = [1, 2, 3, 4];
  const temp = map((a) => delayI(a * a), list);
  console.log(temp); // Promise {<pending>}
  const res = await temp;
  console.log(res);
}
f3();
```

### async:await 중요한 특징

- 결과를 외부로 리턴을 해주고, 해당 값을 출력하고자 할 때 프로미스가 출력된다

```javascript
async function f4() {
  const list = [1, 2, 3, 4];
  const res = await map((a) => delayI(a * a), list);
  //   console.log(res);
  return res;
}

f4(); // Promise {<pending>}
console.log(f4()); // Promise
f4().then(console.log); // [1, 4, 9, 16]
go(f4(), console.log); // [1, 4, 9, 16]
(async () => {
  console.log(await f4());
})(); // [1, 4, 9, 16]
```

- await map을 풀어서 전달하는 것과 풀지 않고 전달하는 것이 동일하다.
- async 함수 리턴 값은 결국 Promise이기 때문이다.

```javascript
async function f5() {
  const list = [1, 2, 3, 4];
  return map((a) => delayI(a * a), list);
}
(async () => {
  console.log(await f5());
})(); // [1, 4, 9, 16]

// 이 말은 f5에 async 키워드 필요없다는 뜻

function f5() {
  return map((a) => delayI(a * a), [1, 2, 3, 4]);
}
(async () => {
  console.log(await f5());
})();
```

<br />
<br />

## 이제 비동기는 async:await로 제어할 수 있는데 왜 파이프라인이 필요한지?

### 서로 해결하고자 하는 문제가 서로 다르다

- 파이프라인
  - 비동기 프로그래밍이 아니라, 명령형 프로그래밍을 하지 않고 안전하게 `함수 합성`을 하기 위함이다.
  - 어떠한 코드를 리스트로 다루면서 연속적인 함수 실행과 합성을 통해 더 효과적으로 함수를 조합하고 로직을 테스트하기 쉽고 유지보수하기 쉽게 만든다.
- async:await
  - 원래 표현식으로 갇혀 있는, 즉 Promise.then.then 식으로 로직 작성이 어렵다 보니 문장으로 다루기 위하여, async:await 키워드를 통해 특정 부분에서 함수 체인이 아닌 문장으로 다루기 위함이다.
  - 따라서 함수 합성이 아니라 오히려 `함수 분해`를 하기 위함이다.
  - 비동기 상황을 동기적 문장으로 풀어 코딩하고 싶을 때 사용한다.

<br />

- 파이프라인으로 코딩했을 때도, 동기적 비동기적 코드가 동일해서 마치 파이프라인으로 코딩하는 이유가 비동기적인 상황을 동기적으로 표현하기 위해서만이라고 생각할 수 있다.
- 파이프라인은 안전하게 비동기 상황을 끝까지 놓치지 않고 연결하거나, 동기 상황인지 비동기 상황인지 상관 없이 합성을 안전하게 하기 위함에 목적이 있다.

```javascript
async function delayI(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 100));
}

function f6(list) {
  return go(
    list,
    L.map((a) => delayI(a * a)), // 비동기
    L.filter((a) => delayI(a % 2)), // 비동기
    L.map((a) => delayI(a + 1)), // 비동기
    take(3),
    reduce((a, b) => delayI(a + b)) // 비동기
  );
}
// 비동기 상황임에도 불구하고 동기적으로 async:await나 Promise가 눈에 보이지 않고,
// callback이나 then 메서드가 보이지 않게 완전히 동기적으로 코드가 표현이 되었다

go(f6([1, 2, 3, 4, 5, 6, 7, 8]), console.log); // 38
```

- 하지만, 위 코드는 async:await을 쓰지 않기 위한 이유가 아니라, 복잡한 for, if문 등을 쉽고 안전하고 간단하게 코딩하기 위함이다.
- f6()에서 파이프라인적 사고가 해결한 방법은, 명령형적인 코드를 해결하기 위한 부분이다.

### 같은 로직을 만약 명령형적으로 작성하고 async:await을 풀었을 경우

- async:await은 비동기 상황을 문장으로 풀어서 해결하기 위해 사용하는 것이다.

```javascript
async function delayI(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 100));
}

async function f7(list) {
  let temp = [];
  for (const a of list) {
    // b가 Promise로 전달되기 때문에 await로 풀어줘야 작업이 가능해진다
    const b = await delayI(a * a);
    // 마찬가지로 정상 동작을 위해 await을 붙여준다
    if (await delayI(b % 2)) {
      // 마찬가지로 정상 동작을 위해 await을 붙여준다
      const c = await delayI(b + 1);
      temp.push(c);
      if (temp.length == 3) break;
    }
  }
  console.log(temp); // [2, 10, 26]
  let res = temp[0],
    i = 0;
  while (++i < temp.length) {
    // 마찬가지로 정상 동작을 위해 await을 붙여준다
    res = await delayI(res + temp[i]);
  }
  return res;
}
go(f7([1, 2, 3, 4, 5, 6, 7, 8]), console.log); // 38
```

- 파이프라인이 해결하는 부분이 await 부분이 아니다.

### 만약 delayI() 함수가 프로미스가 아닌 동기적으로 a를 리턴하는 함수였다면?

- f6 함수는 정상동작하지만, f7 함수는 Promise가 출력된다.
- 전혀 함수들에서 비동기가 일어나지 않는다 하더라도 f7 함수가 async 함수이기 때문이다.

<br />

- f6 함수와 f7 함수는 해결하고자하는 문제 자체가 다르다.
- f6 함수는 async:await을 안쓰기 위해 코딩한 것이 아니다.
- 시간복잡도는 f6 함수와 f7 함수 모두 동일하다 (필요한 부분만 연산)
- 파이프라인은 만들어둔 함수를 잘 가져다 쓰면 되기 때문에 잘 될거라는 기대를 빠르게 가질 수 있는 장점이 있다.
- 파이프라인 사고 방식은 Lazy, Concurrency를 이용하여 최대한 빠르게 실행될 수 있도록 처리할 수 있다. (f7 함수에 Lazy, Concurrency를 적용하는 것은 매우 복잡하다)

<br />
<br />

## async:await와 파이프라인을 같이 사용하기도 하는지?

- 같이 사용하면 편하고 장점이 있다.
- 여러 문장으로 만든 후 결과들을 이용하여 다른 처리를 해줄 경우

```javascript
async function delayI(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 100));
}

async function f8(list) {
  const r1 = await go(
    list,
    L.map((a) => delayI(a * a)), // 비동기
    L.filter((a) => delayI(a % 2)), // 비동기
    L.map((a) => delayI(a + 1)), // 비동기
    take(3),
    reduce((a, b) => delayI(a + b)) // 비동기
  );
  const r2 = await go(
    list,
    L.map((a) => delayI(a * a)), // 비동기
    L.filter((a) => delayI(a % 2)), // 비동기
    reduce((a, b) => delayI(a + b)) // 비동기
  );
  const r3 = await delay1(r1 + r2);
  return r3 + 10;
}

go(f8([1, 2, 3, 4, 5, 6, 7, 8]), (a) => console.log(a, 'f8')); // 106 "f52"
```
