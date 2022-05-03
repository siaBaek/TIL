> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## 리스트 순회

- ES6 되면서 `리스트 순회`가 굉장히 크게 변화했다.

### 1. 기존 ES5 리스트 순회

```javascript
// 배열
const list = [1, 2, 3]
for (var i = 0; i < list.length; i++) {
  console.log(list[i])
}

// 유사 배열
const str = 'abc'
for (var i = 0; i < str.length; i++) {
  console.log(str[i])
}
```

### 2. ES6 리스트 순회

```javascript
// 배열
const list = [1, 2, 3]
for (const a of list) {
  console.log(a)
}
// 유사 배열
const str = 'abc'
for (const a of str) {
  console.log(a)
}
```

<br />

- `for...of`문을 이용하여 간결하게 변경되었다.
- 단순히 복잡한 for문을 간결하게 만든 것을 넘어서, 개발자에게 어떤 규약을 열어주었고, 어떻게 순회에 대해 추상화했고, 어떻게 사용하도록 했는지에 대해 상세히 알아보자.
- [더 알아보기](https://github.com/siaBaek/TIL/blob/main/frontend/javascript/이터러블&이터레이터_프로토콜.md)
