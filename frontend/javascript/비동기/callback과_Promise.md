> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 자바스립트에서 비동기 동시성 프로그래밍 방법

- callback 패턴
- Promise 기반
  - Promise 메서드 체인을 통한 함수 합성
  - async await와 함께 사용하는 방법

## callback과 Promise 비교

- add10, add20 함수를 의도적으로 비동기 상황으로 만들고 딜레이를 만들어 결과를 1000ms 이후에 전달
- 가장 큰 차이는 Promise 패턴에는 `return`이 존재
- 연속적인 실행 시 차이가 눈에 띄게 나게 된다.

### callback

```javascript
function add10(a, callback) {
  setTimeout(() => callback(a + 10), 1000);
}

// add10이 된 이후에 할 일들을 콜백함수를 통해 정의
add10(5, (res) => {
  console.log(res);
});

// 연속 실행
add10(5, (res) => {
  add10(res, (res) => {
    add10(res, (res) => {
      console.log;
    });
  });
});
```

### Promise

```javascript
function add20(a) {
  return new Promise((resolve) => setTimeout(() => resolve(a + 20), 1000));
}

// add20이 된 이후에 할 일들을 then을 이용하여 작성
add20(5).then(console.log);

// 연속 실행
add20(5).then(add20).then(add20).then(console.log);
```
