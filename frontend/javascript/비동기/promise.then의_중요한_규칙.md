> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## promise.then의 중요한 규칙

- then 메서드를 통해 결과를 꺼냈을 때 값이 반드시 프로미스가 아니라는 규칙
- 프로미스 체인이 연속적으로 대기가 걸려있어도 원하는 곳에서 한 번의 then으로 해당하는 결과를 받을 수 있다.
- 이 규칙은 개발자-개발자간, 개발자-언어간 자바스크립트에서 소통하는 법칙이 된다.
- then으로 꺼낸 결과는 아무리 깊이 프로미스를 대기 했어도 반드시 안쪽에 있는 값을 꺼낸다

```javascript
// Promise가 중첩되어 선언되어 있더라도 한번의 then으로 결과를 꺼내볼 수 있다.
Promise.resolve(Promise.resolve(Promise.resolve(1))).then(console.log); // 1

new Promise((resolve) => resolve(new Promise((resolve) => resolve(1)))).then(console.log); // 1
```
