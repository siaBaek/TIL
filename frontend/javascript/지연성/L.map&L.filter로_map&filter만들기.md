> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## L.map, L.filter로 map과 filter 만들기

- 지연적으로 동작하지 않는, 즉시 모든 값을 평가해서 결과를 만드는 map을 L.map와 take를 통해 만들어보자.
-
- 값을 평가하는 것을 미루지 않는 filter를 L.filter와 take를 통해 만들어보자.

```javascript
const takeAll = take(Infinity);

// const map = curry((f, iter) => go(L.map(f, iter), takeAll));

const map = curry(pipe(L.map, takeAll));

console.log(
  map((a) => a + 10),
  L.range(4)
);

// const filter = curry((f, iter) => go(L.filter(f, iter), takeAll));

const filter = curry(pipe(L.filter, takeAll));

console.log(
  filter((a) => a % 2),
  L.range(4)
);
```
