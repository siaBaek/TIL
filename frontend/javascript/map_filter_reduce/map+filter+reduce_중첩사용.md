> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## map, filter, reduce 중첩으로 사용해보기

### 25살 이하 사람들의 나이의 합 구하기

```javascript
const map = (f, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(f(a));
  }
  return res;
};

const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};

const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

const attendance = [
  {name: 'john', age: 20},
  {name: 'paul', age: 25},
  {name: 'james', age: 20},
  {name: 'sasha', age: 30},
  {name: 'susan', age: 27},
];

// 1. 나이를 뽑는 map함수 만들기
console.log(map((a) => a.age, attendance));
// (5) [20, 25, 20, 30, 27]

// 2. 25살 이하의 사람만 뽑기
// map이 받는 attendance를 축약해주기
console.log(
  map(
    (a) => a.age,
    filter((a) => a.age < 25, attendance)
  )
);
// (2) [20, 20]

// 3. 걸러진 데이터를 모두 더하기
const add = (a, b) => a + b;
console.log(
  reduce(
    add,
    map(
      (a) => a.age,
      filter((a) => a.age < 25, attendance)
    )
  )
);
// 40

// (다른 방법) map으로 나이만 뽑고, filter로 거른 후, reduce로 모두 더하기
console.log(
  reduce(
    add,
    filter(
      (a) => a >= 25,
      map((a) => a.age, attendance)
    )
  )
);
// 82
```

<br />

### 함수형 사고

1. add로 reduce한 값을 로그로 출력하고 싶다 (reduce에 iter부분이 데이터가 숫자가 들어있는 배열로 평가될 것을 기대)

```javascript
console.log(reduce(add));
```

2. map을 통해 숫자들이 들어있는 배열을 평가시킬 코드를 작성 (map에서 순회할 배열이, 특정 조건으로 filter된 배열로 평가될 것을 기대)

```javascript
console.log(
  reduce(
    add,
    map((a) => a.age, attendance)
  )
);
```

3. filter를 통해 특정 조건에 맞는 데이터를 뽑아낸다.

```javascript
console.log(
  reduce(
    add,
    map(
      (a) => a.age,
      filter((a) => a.age < 25, attendance)
    )
  )
);
```
