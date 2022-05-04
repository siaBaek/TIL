> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## reduce

- 이터러블 값을 하나의 값으로 축약하는 함수
- 재귀적으로 실행하여 하나의 값 도출하는 구조
- 내부값에 대한 다형성은 보조 함수를 통해 지원해준다. ex) [1,2,3,4,5]
- 외부의 경우에는 이터러블 프로토콜을 따르는 것을 통해 다형성을 지원해준다. ex) attendance
- [reduce](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

```javascript
const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 특정 값을 순회하면서 하나의 값으로 누적해야할 때
const nums = [1, 2, 3, 4, 5];

// 명령형 프로그래밍
let total = 0;
for (const n of nums) {
  total = total + n;
}
console.log(total);

// 함수형 프로그래밍
const reduce = (f, acc, iter) => {
  // 초기값 생략 시(3번째 인자가 없을 시) 첫 번째 value를 초기값으로
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const add = (a, b) => a + b;

console.log(reduce(add, 0, [1, 2, 3, 4, 5])); // 15
console.log(add(add(add(add(add(0, 1), 2), 3), 4), 5)); // 15
// reduce() 메서드에서 초기값 생략시 동작 방법
// reduce(add, [1,2,3,4,5])로 실행하면, reduce(add, 1, [2,3,4,5])로 받은 것처럼 실행하게 된다.
console.log(reduce(add, [1, 2, 3, 4, 5])); // 15

// reduce같은 경우 보조함수를 통해 어떻게 축약할지 완전 위임 -> 단순 숫자 배열 외에도 복잡한 데이터도 가능
// attendance의 모든 나이를 더하는 로직을 reduce만으로 작성해보기
console.log(reduce((total_age, person) => total_age + person.age, 0, attendance)); // 122
```
