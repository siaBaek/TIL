> 인프런 유인동 지식공유자님의 `함수형 프로그래밍과 JavaScript ES6+` 강의를 참고하였습니다

## map, filter 계열 함수들이 가지는 결합 법칙

- 특정 방식으로 평가 순서를 다르게 바꿔도 동일한 결과를 만든다.

<br />

- 사용하는 데이터가 무엇이든지
- 사용하는 보조 함수가 순수 함수라면 무엇이든지
- 아래와 같이 결합한다면 둘 다 결과가 같다

```
[[mapping, mapping], [filtering, filtering], [mapping, mapping]]
=
[[mapping, filtering, mapping], [mapping, filtering, mapping]];
```

- 즉시 평가하지 않고 지연 평가하는 성질을 가져 아래와 같이 작성하더라도 결과가 같다.
- take 같이 결과를 꺼내는 함수를 통해 즉시 평가되는 값을 통해 꺼내든, 지연 평가되는 값을 통해 꺼내든 완전히 같은 결과이다.
