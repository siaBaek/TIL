> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## yield\*

```javascript
L.flatten = function* (iter) {
  for (const a of iter) {
    if (isIterable(a)) for (const b of a) yield b;
    else yield a;
  }
};
```

- yield\*을 활용하면 위 코드를 아래와 같이 변경할 수 있다.
- `yield* iterable`은 `for(const val of iterable) yield val` 와 같다.

```javascript
L.flatten = function* (iter) {
  for (const a of iter) {
    if (isIterable(a)) yield* a;
    else yield a;
  }
};
```

## L.deelFlat

- 깊은 iterable을 모두 펼쳐주는 함수

```javascript
L.deepFlat = function* f(ter) {
  for (const a of iter) {
    if (isIterable(a)) yield* f(a);
    else yield a;
  }
};
console.log([...L.deepFlat([1, [2, [3, 4, [[5]]]]])]);
// [1, 2, 3, 4, 5]
```
