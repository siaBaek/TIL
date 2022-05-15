> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## callback과 Promise의 차이

- 가장 중요한 차이는 어떻게 결과를 꺼내어 보는가가 아니다.
- Promise가 콜백 지옥을 then 으로 해결하는 것에 초점이 맞춰지는 경향이 있다.
- 정말 특별하게 콜백과 다른 차이를 가지는 점은 then 메서드를 이용해 결과를 꺼내본다는 것이 아니라 비동기 상황을 일급값으로 다룬다는 것
- Promise 클래스를 통해 만들어진 인스턴스를 반환하는데, 프로미스는 `대기, 성공, 실패`를 다루는 일급값으로 이루어져 있다.

<br />
<br />

- 콜백함수는 비동기 상황을 코드로만 표현하지만,
- 프로미스는 비동기 상황에 대한 값을 만들어서 `리턴`을 한다는 점이 굉장히 큰 차이이다.

```javascript
var a = add10(5, (res) => {
  add10(res, (res) => {
    add10(res, (res) => {
      console.log;
    });
  });
});
console.log(a); // undefined
// 실행 후 어떠한 일도 할 수 없다.

var b = add20(5).then(add20).then(add20).then(console.log);
console.log(b); // Promise {<pending>}
// 리턴된 프로미스를 가지고 이후에 원하는 일들을 또 할 수 있다.
```

- 프로미스가 리턴되기 때문에 계속해서 then을 통해 연결지어 일을 할 수 있다.
- 비동기로 일어난 상황에 대해서 값으로 다룰 수 있다.
- 비동기 상황이 일급값으로서 다뤄지는 상황(일급값: 어떤 변수에 할당될 수 있고, 함수에 전달될 수 있고, 계속해서 작업을 이어나갈 수 있다.)

## 비동기 상황을 일급값으로 리턴하는 Promise (일급 활용)

- 함수에게 전달할 수 있고, 해당 값이 어떤 값인지 Promise인지 아닌지 등을 확인할 수 있다.
- 이를 활용하여 함수를 만들어보자

```javascript
// a,f: 동기적으로 동작하는 값, 함수여야 한다
const go1 = (a, f) => f(a);
const add5 = (a) => a + 5;
console.log(go1(10, add5)); // 15
```

- 만약 a가 어느 정도 시간이 지난 후 알 수 있는 값이라면?

```javascript
console.log(go1(Promise.resolve(10), add5)); // [object Promise]5
// 위 코드를 정상적으로 동작하도록 하고 싶음

const delay100 = (a) => new Promise((resolve) => setTimeout(() => resolve(a), 100));

const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
const add5 = (a) => a + 5;

console.log(go1(delay100(10), add5)); // Promise {<pending>}

var r = go1(10, add5);
console.log(r); // 15
var r2 = go1(delay100(10), add5);
r2.then(console.log); // 100ms 후 15
```

- 표현을 완전히 똑같이 해보기

```javascript
const delay100 = (a) => new Promise((resolve) => setTimeout(() => resolve(a), 100));

const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
const add5 = (a) => a + 5;

const n1 = 10;
go1(go1(n1, add5), console.log); // 15

const n2 = delay100(10);
go1(go1(n2, add5), console.log); // 100ms 후 15
console.log(go1(go1(n2, add5), console.log)); // Promise {<pending>}
```
