> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 지연 평가 + Promise의 효율성

- 지연 평가와 즉시 평가의 map, filter, reduce 에서 동기와 비동기적인 상황을 모두 원하는대로 제어하도록 코드 완성되었다.

### L.map에서 비동기적인 상황이 있었다면?

- 비동기적인 작업이 함수 대기열에 있는 상황에서, 지연 평가를 통해 동시성 프로그래밍을 제어하기 때문에 L.map 내부 코드에 들어오지 않게 된다(실행되지 않게 된다)

```javascript
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => new Promise((resolve) => setTimeout(() => resolve(a * a), 1000))),
  L.filter((a) => a % 2),
  take(5), // 앞에서 5개만
  console.log
); // [1, 9, 25, 49]
// 아래 코드가 위 코드보다 처리 속도가 훨씬 빠르다
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => new Promise((resolve) => setTimeout(() => resolve(a * a), 1000))),
  L.filter((a) => a % 2),
  take(2), // 앞에서 2개만
  console.log
); // [1, 9]
```

<br />

### L.filter 에서도 비동기적인 상황이 있었다면?

- 필요한 상황의 값만 들어오기 때문에 훨씬 효율적으로 동작하게 된다.

```javascript
go(
  [1, 2, 3, 4, 5, 6, 7, 8],
  L.map((a) => {
    console.log(a);
    return new Promise((resolve) => setTimeout(() => resolve(a * a), 1000));
  }),
  L.filter((a) => {
    console.log(a);
    return new Promise((resolve) => setTimeout(() => resolve(a % 2), 1000));
  }),
  take(2),
  console.log
);
// 1
// 1 (filter에서 true이므로 res에 추가된다)
// 2
// 4 (filter에서 false이므로 res에 추가되지 않는다)
// 3
// 9 (filter에서 true이므로 res에 추가된다) -> take(2)로 인해 더 이상 진행되지 않고 console.log가 실행된다
// [1, 9]
```
