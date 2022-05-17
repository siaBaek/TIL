> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 즉시 병렬적으로 평가하기 - C.map, C.filter

- 함수열 쭉 만들고, 마지막에 평가짓는 함수인 reduce, take에서 지금까지 등록된 함수를 병렬적/동기적으로 실행하는 것을 선택하는 것으로 여러 개의 대기열을 다루는 식으로 함수열을 다루었다.

### C.map, C.filter

- 특정 함수 라인에서만 병렬적으로 평가하고 그 이후엔 동기적으로 평가하고 싶을 때

```javascript
// map은 L.map과 takeAll로 만들어져 있고,
// filter는 L.filter와 takeAll로 만들어져 있다.

C.takeAll = C.take(Infinity);

C.map = curry(pipe(L.map, C.takeAll));

C.filter = curry(pipe(L.filter, C.takeAll));

const delay1000 = (a) =>
  new Promise((resolve) => {
    console.log('hi');
    setTimeout(() => resolve(a)), 1000;
  });

C.map((a) => delay1000(a * a), [1, 2, 3, 4]).then(console.log);
// <4> hi
// [1, 4, 9, 16]

C.filter((a) => delay1000(a % 2), [1, 2, 3, 4]).then(console.log);
// <4> hi
// [1, 3]
```
