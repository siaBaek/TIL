## 동기 상황에서 에러 핸들링은 어떻게 해야하는지?

```javascript
function f1(list) {
  return list
    .map((a) => a + 10)
    .filter((a) => a % 2)
    .slice(0, 2);
}
// 배열 전달
console.log(f1([1, 2, 3])); // [11, 13] 정상 작동

// 빈 배열 전달
console.log(f1([])); // [] 문제없이 흘려보내는 식으로 동작

// null 전달
console.log(f1(null)); // Error

// 해결방법 1: 전달되지 않을 경우 디폴트 값 []로 설정
function f1(list = []) {
  return list
    .map((a) => a + 10)
    .filter((a) => a % 2)
    .slice(0, 2);
}

// 해결방법 2: null 등 쌩뚱맞은 값이 올 경우 []로 설정
function f1(list) {
  return (list || [])
    .map((a) => a + 10)
    .filter((a) => a % 2)
    .slice(0, 2);
}
```

- 에러 핸들링은 언어마다 다른 특징, 선호하는 방식, 상황에 따라 다르다.
- 에러가 생겼다는건, 값이 엉망이라는 것 -> 최대한 비슷한 값을 리턴하거나 그냥 에러를 출력하거나

### try catch 문으로 에러 핸들링하기

- 만약 에러 핸들링해서 최대한 비슷하게 안전하게 흘려보내고 싶다면

```javascript
function f2(list) {
  try {
    return list
      .map((a) => a + 10)
      .filter((a) => a % 2)
      .slice(0, 2);
  } catch (e) {
    return [];
  }
}
cosnole.log(f2(null)); // []
cosnole.log(f2([])); // []
```

- 만약 map함수 내부에서 에러가 났다면

```javascript
function f3(list) {
  try {
    return list
      .map((a) => asdf)
      .filter((a) => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e);
    return [];
  }
}
cosnole.log(f3([1, 2, 3]));
// ReferenceError: asdf is not defined
// []

function f4(list) {
  try {
    return list
      .map((a) => JSON.parse(a))
      .filter((a) => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e);
    return [];
  }
}
cosnole.log(f4(['0', '1', '2', '{']));
// SyntaxError: Unexpected end of JSON input
// []
```

<br />
<br />

## 비동기 상황에서 에러 핸들링은 어떻게 해야하는지?

- 에러 핸들링이 동기 상황보다 조금 더 어렵다.

<br />

```javascript
function f5(list) {
  try {
    return list
      .map(
        (a) =>
          new Promise((resolve) => {
            asdf;
          })
      )
      .filter((a) => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e); // 에러 출력안됨
    return [];
  }
}
console.log(f5(['0', '1', '2', '{'])); // []
// 빈 배열은 catch문에서 나온 것이 아니라, map에서 엉뚱한 결과로서 계산된 결과이다
```

- 에러 핸들링이 되지 않고 있다.
- 프로미스를 잘 제어해 줄 수 있는 상태의 함수가 아니기 때문에 catch로 넘어가지 않았다

```javascript
function f6(list) {
  try {
    return list
      .map(
        (a) =>
          new Promise((resolve) => {
            resolve(JSON.parse(a));
          })
      ) // 내부에서 에러가 났으면 rejected 상태가 되어야 한다
      .filter((a) => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e);
    return [];
  }
}

f6(['0', '1', '2', '{'])
  .then(console.log)
  .catch((e) => {
    console.log('에러 핸들링');
  }); // 출력 안됨
```

<br />
<br />

## 동기/비동기 에러 핸들링에서의 파이프라인의 이점은?

- 비동기 상황을 잘 정리하고 있는 코드로 같은 작업을 해보자

```javascript
async function f7(list) {
  try {
    return go(
      list,
      map(
        (a) =>
          new Promise((resolve) => {
            resolve(JSON.parse(a));
          })
      ),
      filter((a) => a % 2),
      take(2)
    );
  } catch (e) {
    console.log(e); // 이건 출력이 안된다
    return [];
  }
}

f7(['0', '1', '2', '{'])
  .then(console.log)
  .catch((e) => {
    console.log('에러 핸들링');
  }); // 에러 핸들링
// 에러 발생할 경우 함수 내부 catch로는 가지 않지만, 호출 시  catch로는 이동
// 함수 내부 catch로 가려면 Promise rejected로 평가 되어야 한다
```

```javascript
async function f8(list) {
  try {
    return await go(
      // await 추가
      list,
      map(
        (a) =>
          new Promise((resolve) => {
            resolve(JSON.parse(a));
          })
      ),
      filter((a) => a % 2),
      take(2)
    );
  } catch (e) {
    console.log(e); // 출력이 된다
    return [];
  }
}

f8(['0', '1', '2', '{']).then(console.log);
// SyntaxError: Unexpected end of JSON input
// []
```

```javascript
async function f9(list) {
  try {
    // Promise rejected으로 평가 되려면 res 값이 프로미스여야한다는 것
    const res = go(
      list,
      map(
        (a) =>
          new Promise((resolve) => {
            resolve(JSON.parse(a));
          })
      ),
      filter((a) => a % 2),
      take(2)
    );
    return await res;
  } catch (e) {
    console.log(e); // 출력이 된다
    return [];
  }
}

f9(['0', '1', '2', '{']).then(console.log);
// SyntaxError: Unexpected end of JSON input
// []
```

- 위 코드의 조건은 함수 합성이 연속적으로 잘 되어야 한다는 것
- 잘 연결 되어 있고 잘 결론 지어져 있어야 하고, 그게 아니라면 쉽게 에러 핸들링을 할 수가 없다.

```javascript
async function f10(list) {
  try {
    // Promise rejected으로 평가 되려면 res 값이 프로미스여야한다는 것
    const res = go(
      list,
      map(
        (a) =>
          new Promise((resolve) => {
            // 즉시 모두 평가
            console.log(a); // 0, 1, 2, 3, 4, {
            resolve(JSON.parse(a));
          })
      ),
      filter((a) => a % 2),
      take(2)
    );
    return await res;
  } catch (e) {
    console.log(e);
    return [];
  }
}

f10(['0', '1', '2', '3', '4', '{']).then(console.log);
// SyntaxError: Unexpected end of JSON input
// []
// map 함수에서

async function f11(list) {
  try {
    // Promise rejected으로 평가 되려면 res 값이 프로미스여야한다는 것
    const res = go(
      list,
      L.map(
        (a) =>
          new Promise((resolve) => {
            console.log(a); // 0, 1, 2, 3
            resolve(JSON.parse(a));
          })
      ),
      L.filter((a) => a % 2),
      take(2)
    );
    return await res;
  } catch (e) {
    console.log(e);
    return [];
  }
}
// map, filter에서 결과를 미루면 에러 나기 전에 연산이 끝나기 때문에 문제 없이 결과를 꺼낼 수 있다

f11(['0', '1', '2', '3', '4', '{'])
  .then(console.log)
  .catch((e) => {
    console.log('에러 핸들링');
  });
// [1, 3]
// 호출 시 catch문 출력 안됨
```

- 에러 핸들링을 할 때도 await에 Promise를 잘 걸어줘야 하는 것이고, Promise를 await에 잘 걸어주기 위해서는 함수 안쪽에서 나는 에러들이 중간에 에러가 나도 전체 Promise에게 Kleisli 방식으로 안전하게 합성해서 뒤로 연결해서 보내줄 수 있도록 준비가 되어 있어야 안전한 에러 핸들링이 가능하다.
- 중간에 트랜잭션 처리를 하더라도, insert, update 등을 연속 실행하다가 에러가 났을 때 catch에서 뽑아서 rollback을 해주는 것을 얼마든지 할 수 있다.

<br />
<br />

## 마무리

```text
지금까지 최신 자바스크립트의 기법들인 이터러블/이터레이러/제너레이터/Promise/구조 분해/전개 연산자 등을 이용하여 자바스크립트의 기본 기능 위에서 함수형 프로그래밍을 하는 기법들을 살펴보았습니다. 이 내용들은 기초적인 내용들이지만 잘 체득하고 응용하면 자바스크립트가 동작하는 환경들에서 성공적인 프로그래밍을 할 수 있다고 생각합니다. 아래는 예제 코드와 참고할만한 링크입니다.
예제 코드: https://github.com/indongyoo/functional-javascript-01
페이스북 '함수형 자바스크립트' 그룹: https://www.facebook.com/groups/539983619537858/
끝까지 강의를 들어주신 분들께 감사드립니다.
```
