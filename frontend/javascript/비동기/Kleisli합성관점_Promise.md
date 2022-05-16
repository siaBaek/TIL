> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## Kleisli Composition(Kleisli Arrow)

- 오류가 있을 수 있는 상황에서의 함수 합성을 안전하게 하는 하나의 규칙
- 들어오는 인자가 완전히 잘못된 인자여서 함수에서 오류가 나는 상황이라던지 정확한 인자가 들어왔더라도 어떤 함수가 의존하고 있는 외부의 상태에 의해 결과를 정확히 전달할 수 없는 상황 등에서 에러가 나는 것을 해결하기 위한 함수 합성
- Promise는 Kleisli Composition을 지원하는 도구

<br />

- 수학적 프로그래밍
  - 항상 정확한 상수/변수를 통한 함수 합성, 평가 및 연산이 되고 결과를 만든다.
- 현대 프로그래밍 => 상태, 효과, 외부 의존성 존재
  - 외부 세상의 상황이나 효과의 상황에 의해 함수 합성이 원하는 대로 정확하게 이루어지지 않을 수 있는 가능성이 항상 존재한다.

```javascript
// f . g 합성
// f(g(x)) 결과는 어느 시점에 평가해도 항상 동일해서,
f(g(x)) = f(g(x)); // 해당 식이 성립한다
```

- 하지만, 실무에서 좌항을 평가할 때의 g가 바라보는 값이 우항을 평가할 때 다른 값으로 변하거나 값이 없어져서 오류가 날 수 있고, 이 때는 f(g(x)) = f(g(x)) 식이 성립하지 않게 된다
- 특정한 규칙을 만들어서 합성을 안전하게 만들고 조금 더 수학적으로 바라볼 수 있게 만들어주는 함수 합성이 `kleisli composition`

### Kleisli composition 규칙

- g와 f를 연속적으로 실행하는 상황에서
- g에서 에러 없이 동작하면, `f(g(x)) = f(g(x))`로 동작하지만
- g에서 에러가 발생한 경우, `f(g(x)) = g(x)`로 동작한다

```javascript
var users = [
  {id: 1, name: 'aa'},
  {id: 2, name: 'bb'},
  {id: 3, name: 'cc'},
]; // 상태: 유저가 담긴 Array

const getUserById = (id) => find((u) => u.id == id, users);

const f = ({name}) => name;
const g = getUserById;

// 먼저 g에게 id를 전달하여 유저를 찾고, 유저의 이름을 추출해서 리턴
const fg = (id) => f(g(id));

console.log(fg(2)); // bb
console.log(fg(2) == fg(2)); // true
```

- 하지만 실제 프로그래밍에서는, users의 상태가 변하기도 한다.
- f라는 함수는 항상 name을 가진 객체를 받았을 때만 정상 동작하는 함수
- g라는 함수는 해당 결과가 있다고 가정했을 때만 정상 동작하는 함수

```javascript
var users = [
  {id: 1, name: 'aa'},
  {id: 2, name: 'bb'},
  {id: 3, name: 'cc'},
]; // 상태: 유저가 담긴 Array

const getUserById = (id) => find((u) => u.id == id, users);

const f = ({name}) => name;
const g = getUserById;

// 먼저 g에게 id를 전달하여 유저를 찾고, 유저의 이름을 추출해서 리턴
const fg = (id) => f(g(id));

const r1 = fg(2);

// pop
users.pop();
users.pop();
console.log(users); // [{id: 1, name: 'aa'}]

const r2 = fg(2); // Error
```

- 만약 pop이 일어나지 않고 f와 g에 항상 1~3의 값만 전달된다고 하면 항상 안전한 합성이 가능하겠지만, 원하는대로 정확하게 이루어지지 않을 가능성이 존재한다.
- 이를 항상 안전한 합성이 가능하도록 하는 것이 kleisli composition

### 1. fg 함수 변경

```javascript
var users = [
  {id: 1, name: 'aa'},
  {id: 2, name: 'bb'},
  {id: 3, name: 'cc'},
]; // 상태: 유저가 담긴 Array

const getUserById = (id) => find((u) => u.id == id, users);

const f = ({name}) => name;
const g = getUserById;

// const fg = (id) => f(g(id));
const fg = (id) => Promise.resolve(id).then(g).then(f);

fg(2).then(console.log); // bb

users.pop();
users.pop();
console.log(users); // [{id: 1, name: 'aa'}]

fg(2).then(console.log); // Promise {<rejected>: TypeErrer:...}
```

### 2. 프로미스 reject 처리 (getUserById에서 or 처리)

```javascript
var users = [
  {id: 1, name: 'aa'},
  {id: 2, name: 'bb'},
  {id: 3, name: 'cc'},
]; // 상태: 유저가 담긴 Array

const getUserById = (id) => find((u) => u.id == id, users) || Promise.reject('없어요!');

const f = ({name}) => name;
const g = getUserById;

// const fg = (id) => f(g(id));
const fg = (id) => Promise.resolve(id).then(g).then(f);

console.log(g(1)); // {id: 1, name: "aa"}
console.log(g(2)); // {id: 2, name: "bb"}

users.pop();
users.pop();

console.log(g(1)); // {id: 1, name: "aa"}
console.log(g(2)); // Promise {<rejected>: "없어요!"}

fg(2).then(console.log); // Promise {rejected>: "없어요!"}
// g만 실행한 것처럼 동일한 상황으로 진행된다

// 만약 아래와 같이 함수 합성 했을 경우에는 엉뚱한 결과를 쓸 수도 있게 된다
f(g(2)); // undefined
```

- 콘솔에 없어요!를 바로 출력하려면 fg 함수에 catch문으로 에러 내용을 리턴해준다

```javascript
const fg = (id) =>
  Promise.resolve(id) // 에러발생!
    .then(g) // 실행안함
    .then(f) // 실행안함
    .catch((a) => a); // 없어요! 리턴
fg(2).then(console.log); // 없어요!
```
