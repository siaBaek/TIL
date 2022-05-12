> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## L.flatten, flatten

- 배열 안에 배열을 모두 펼쳐 하나의 배열로 만들어주는 함수

```javascript
console.log([[1, 2], 3, 4, [5, 6], [7, 8, 9]]);

const isIterable = (a) => a && a[Symbol.iterator];

const L = {};
L.flatten = function* (iter) {
  for (const a of iter) {
    if (isIterable(a)) {
      for (const b of a) yield b;
    } else yield a;
  }
};

var it = L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]);
console.log([...it]); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log(take(3, L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]))); // [1, 2, 3]
```

### L.flatten으로 즉시평가하는 faltten 함수 만들기

```javascript
const flatten = pipe(L.flatten, take(Infinity));
console.log(flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]])); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
