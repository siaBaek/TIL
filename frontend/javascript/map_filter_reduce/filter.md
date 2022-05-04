> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## filter

- filter 함수 역시 이터러블 프로토콜을 따르고 함수형 프로그래밍을 통해 구현해 중복을 제거하고 구현할 수 있다.
- 내부값에 대한 다형성은 보조 함수를 통해 지원해준다. ex) [1,2,3,4]
- 외부의 경우에는 이터러블 프로토콜을 따르는 것을 통해 다형성을 지원해준다. ex) attendance
- [filter](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

```javascript
// 특정 나이 이하, 이상을 걸러내는 로직을 종종 사용
const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 명령형 프로그래밍
let under25 = [];
for (const a of attendance) {
  if (a.age < 25) under25.push(a);
}
console.log(...under25);
// {name: 'john', age: 20}
// {name: 'james', age: 20}

let over25 = [];
for (const a of attendance) {
  if (a.age >= 25) over25.push(a);
}
console.log(...over25);
// {name: 'paul', age: 25}
// {name: 'sasha', age: 30}
// {name: 'susan', age: 27}

// 함수형 프로그래밍
const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};
console.log(...filter((a) => a.age < 25, attendance));
// {name: 'john', age: 20}
// {name: 'james', age: 20}
console.log(...filter((a) => a.age >= 25, attendance));
// {name: 'paul', age: 25}
// {name: 'sasha', age: 30}
// {name: 'susan', age: 27}
console.log(filter((n) => n % 2, [1, 2, 3, 4]));
// (2) [1, 3]

console.log(
  filter(
    (n) => n % 2,
    (function* () {
      yield 1;
      yield 2;
      yield 3;
      yield 4;
      yield 5;
    })()
  )
);
// (3) [1, 3, 5]
```
