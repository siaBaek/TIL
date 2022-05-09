> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## L.map

- 평가를 미루는 성질을 가지고 평가 순서를 달리 조작할 수 있는 준비가 되어 있는 이터레이터를 반환하는 제너레이터 함수
- 앞서 만든 [map](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/map_filter_reduce/map.md)을 지연성을 가진 L.map으로 만들어보자
- 제너레이터 이터레이터 프로토콜 기반으로 구현해보자

```javascript
const L = {};
L.map = function* (iter) {
  for (const a of iter) yield a;
};
const it = L.map([1, 2, 3]);
it.next(); // {value: 1, done: false}
```

```javascript
// 우리가 원하는건, 1, 2, 3을 어떤 함수를 적용한 값을 다시 만드려는 것
const L = {};
L.map = function* (f, iter) {
  for (const a of iter) yield f(a);
};
const it = L.map((a) => a + 10, [1, 2, 3]);
it.next(); // {value: 11, done: false}
```

### L.map의 특징

```javascript
const L = {};
L.map = function* (f, iter) {
  for (const a of iter) yield f(a);
};
const it = L.map((a) => a + 10, [1, 2, 3]); // 아무 것도 진행되지 않고, next()를 통해 내가 평가하는 만큼의 값만 얻어올 수 있다.
console.log(it); // L.map {<closed>}
console.log(...it); // 11, 12, 13
```

- 새로운 배열을 만들지 않고 값 하나하나 순회하며 yield를 통해 함수가 적용된 값을 이터레이터의 next()를 실행할 때 마다 하나씩 전달하게 된다.
- `...it, it.next().value`처럼 이터레이터 객체를 원하는 방법으로 평가할 수 있다.
