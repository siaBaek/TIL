> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## L.flatMap, flatMap

- map과 flatten을 동시에 하는 함수

```javascript
// 기본 JS 내장 flatMap 동작방법
console.log(
  [
    [1, 2],
    [3, 4],
    [5, 6],
  ].flatMap((a) => a)
); // [1, 2, 3, 4, 5, 6]

console.log(
  [
    [1, 2],
    [3, 4],
    [5, 6],
  ].flatMap((a) => a.map((a) => a * a))
);
// [1, 4, 9, 16, 25, 36]

// 사실상 map을 한 값에 flatten한 것과 동일하게 동작
// map과 flatten이 결국 비효율적으로 동작하기 때문에 flatMap이 등장
```

- 첫 번째 map을 할 때, 안에 있는 모든 값을 순회하면서 새로운 배열이 만들어진다.
- 그렇게 만든 모든 배열을 만들고, 다시 한번 전체를 순회하면서 다시 배열을 담기 때문에 비효율적이다.
- 따라서 이것을 한 번에 해서 조금 더 효율적으로 동작하게끔 하는 것이다.
- 순회하지 않아도 되는 부분이 있는 코드가 아니라면, 사실 시간복잡도 면에서는 차이가 없다. 순회해야할 것을 모두 순회하는 것이기 때문이다.
- 만약 앞에 있는 3개만 필요하다거나 할 때 조금의 효율이 발생할 수 있다.
- 어레이에만 적용 가능한 flatMap이 아닌, 좀 더 다형성이 높고 지연성이 있는 조금 더 효율성 있는 L.flatMap을 만들어보자

```javascript
// 이미 flatten 구현되어 있기 때문에 단순하게 구현이 가능하다
L.flatMap = pipe(L.map, L.flatten);

var it = L.flatMap(
  map((a) => a * a), // a => a
  [
    [1, 2],
    [3, 4],
    [5, 6, 7],
  ]
);
console.log([...it]); // [1, 4, 9, 16, 25, 36, 49]
```

### L.flatMap으로 flatMap 만들기

```javascript
const flatMap = pipe(L.flatMap, take(Infinity));

// 또는

const flatMap = curry(pipe(L.map, flatten));

console.log(
  flatMap(
    (a) => a,
    [
      [1, 2],
      [3, 4],
      [5, 6, 7],
    ]
  )
);

// 활용

console.log(map(L.range), [1, 2, 3]);
// [Array(1), Array(2), Array(3)]
// 0: [0]
// 1: [0, 1]
// 2: [0, 1, 2]

console.log(flatMap(L.range, [1, 2, 3]));
// [0, 0, 1, 0, 1 ,2]

console.log(
  flatMap(
    L.range,
    map((a) => a + 1, [1, 2, 3])
  )
); // [0, 1, 0, 1, 2, 0, 1, 2, 3]

var it = L.flatMap(
  L.range,
  map((a) => a + 1, [1, 2, 3])
);

console.log(it.next()); // {value: 0, done: false}
console.log(it.next()); // {value: 1, done: false}
console.log(it.next()); // {value: 0, done: false}
console.log(it.next()); // {value: 1, done: false}
console.log(it.next()); // {value: 2, done: false}

console.log(
  take(
    3,
    L.flatMap(
      L.range,
      map((a) => a + 1, [1, 2, 3])
    )
  )
); // [0, 1, 0]
```
