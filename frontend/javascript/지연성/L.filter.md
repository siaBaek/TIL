> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## L.filter

- 이터레이터를 순회하면서 yield하는 경우, yield하는 값이 f를 적용해봤을 때 true로 평가되는 값이 떨어진다면 그 때만 a를 yield하면 된다.
- 평가를 미루는 성질을 가지고 평가 순서를 달리 조작할 수 있는 준비가 되어 있는 이터레이터를 반환하는 제너레이터 함수
- 앞서 만든 [filter](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/map_filter_reduce/filter.md)을 지연성을 가진 L.filter으로 만들어보자
- 제너레이터 이터레이터 프로토콜 기반으로 구현해보자

```javascript
const L = {};
L.filter = function* (f, iter) {
  for (const a of iter) if (f(a)) yield a;
};
var it = L.filter((a) => a % 2, [1, 2, 3, 4]);
it.next(); // {value: 1, done: false}
it.next(); // {value: 3, done: false}
```

- next()할 때마다 yield하는데 총 4번 되는 것이 아니라 원하는 상황에서만 되기 때문에 next()를 여러 번 했을 때, {value: undefined, done: true}가 될 때까지 필터링해서 value가 꺼내질 수 있도록 한다.
