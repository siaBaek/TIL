> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## async/await

- 비동기 상황을 동기적 코드를 다루기 위한 방법 중 하나
- [하스켈](https://ko.wikipedia.org/wiki/%ED%95%98%EC%8A%A4%EC%BC%88)의 do notation과 비슷하다.
- 비동기적으로 일어나는 일들을 문장으로 다룰 때 동기적 문장으로 다룰 수 있도록 하는 키워드

```javascript
function delay(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a), 500));
}

function f() {
  const a = delay(10);
  console.log(a);
}
f(); // Promise {<pending>}

async function f1() {
  const a = await delay(10);
  const b = await delay(5);
  // 프로미스가 결과를 만들고 끝날때까지 멈추고 내부 값이 a에 할당된다
  console.log(a + b);
}
f1(); // 15
```

### async/await의 기반 Promise

- delay같은 함수에서는 결국에는 Promise를 사용한다.
- f2 함수만 보면, Promise 없이 동작하는 것 같지만, 함수 실행할 때 await 하기 위해서는 그 함수가 반드시 Promise를 리턴해야한다.

### delay 함수에서도 Promise 안보이게 하려면

- delayIdentity에서는 Promise가 사라졌지만, 결국 어딘가에는 Promise가 있어야 한다.
- Promise를 리턴하기로 준비되어 있는 라이브러리를 사용만 할 때는 async/await로 대다수 커버할 수 있지만, 내가 Promise를 다뤄서 리턴하거나, 값으로 다루면서 평가 시점을 다루는 코드를 작성할 때는 직접 Promise를 다룰 수 있어야 한다.

```javascript
function delay(time) {
  return new Promise((resolve) => setTimeout(() => resolve(), time));
}

async function delayIdentity(a) {
  await delay(500);
  return a;
}

async function f1() {
  const a = await delayIdentity(10);
  const b = await delayIdentity(5);
  console.log(a + b); // 이 결과가 동기적으로 떨어진다
}
f1();
```

### async는 어떤 값을 리턴하더라도 Promise를 리턴하는 함수이다

```javascript
function delay(time) {
  return new Promise((resolve) => setTimeout(() => resolve(), time));
}

async function delayIdentity(a) {
  await delay(500);
  return a;
}

async function f2() {
  const a = await delayIdentity(10);
  const b = await delayIdentity(5);
  return a + b;
}
console.log(f2()); // Promise {<pending>}

// Promise를 전달하지 않더라도 Promise를 리턴한다
async function f3() {
  const a = 10;
  const b = 5;
  return a + b;
}
console.log(f3()); // Promise {<fulfilled>: 15}
```

- async 함수의 결과를 리턴해서 다른 곳에서 다시 값을 받아서 이어서 프로그래밍하는 식으로 사용하려면 아래와 같이 작성할 수 있다.

```javascript
function delay(time) {
  return new Promise((resolve) => setTimeout(() => resolve(), time));
}

async function delayIdentity(a) {
  await delay(500);
  return a;
}

async function f2() {
  const a = await delayIdentity(10);
  const b = await delayIdentity(5);
  return a + b;
}

f2(); // Promise {<pending>}
console.log(f2()); //
f2().then(console.log); // 15
go(f2(), console.log); // 15
(async () => {
  console.log(await f2());
})(); // 15
```

### await는 즉시 Promise로 평가되는 함수에 사용할 수 있다.

```javascript
const pa = Promise.resolve(10);

(async () => {
  console.log(await pa);
})();
```

```javascript
async function f2() {
  const a = await delayIdentity(10);
  const b = await delayIdentity(5);
  return a + b;
}
const pa = f2();

(async () => {
  console.log(await pa);
  console.log(await pa);
  console.log(await pa);
})();
```
