[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/stack/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/stack/README.ko-KR.md)

## **Stack (스택)**

\- 아래의 두 가지 연산을 가진 요소들의 집합인 추상 자료형

- **push**는 집합에 요소를 추가하는 것이며,
- **pop**은 아직 제거되지 않은 가장 최근에 추가된 요소를 제거하는 연산

\- 요소가 스택에서 나오는 과정은 **LIFO(Last In, First Out, 후입선출)**라는 이름으로 확인이 가능하다

\- 즉, 나중에 들어간 것이 먼저 나오는 구조이다

\- 물건들이 다른 물건들 위에 쌓이게 되는 것에서 유추된 이름

\- 최상단 물건은 빼내기 쉽지만, 깊이 있는 물건을 빼내려면 다른 물건들을 먼저 빼내야 하는 작업이 필요하다

![Stack](https://upload.wikimedia.org/wikipedia/commons/b/b4/Lifo_stack.png)

### 1) 자바스크립트로 구현해보기

\- [Linked List](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/singly_linked_list.md) 이용하기

```javascript
import LinkedList from '../linked-list/LinkedList'; // 이전 게시물 참고

export default class Stack {
  constructor() {
    // 연결 리스트와 구조가 유사하기 때문에 연결 리스트 기반으로 스택을 구현해보자
    // 스택의 push/pop 작업은 연결 리스트의 append/deleteHead 와 비교된다
    this.linkedList = new LinkedList();
  }

  isEmpty() {
    // 연결 리스트에 Head 없다면, 비어 있는 스택
    return !this.linkedList.head;
  }

  peek() {
    if (this.isEmpty()) {
      // 연결 리스트 비어있다면 스택을 수정하지 않고 최상단의 요소에 접근할 수 없다
      return null;
    }
    // 스택을 수정하지 않고 최상단의 요소에 접근할 수 있게
    // 삭제 없이 연결 리스트의 시작부터 값을 읽는다
    return this.linkedList.head.value;
  }

  push(value) {
    // push는 스택의 맨 위에 값을 놓는 것을 의미한다.
    // 따라서 연결 리스트 시작 부분에 새로운 값을 추가한다
    this.linkedList.prepend(value);
  }

  pop() {
    // 연결 리스트에서 첫 번째 노드를 삭제해보자
    // 만약 Head가 없다면(연결 리스트가 비어있다면) null을 리턴
    const removedHead = this.linkedList.deleteHead();
    return removedHead ? removedHead.value : null;
  }

  // 스택 -> Array
  toArray() {
    return this.linkedList.toArray().map((linkedListNode) => linkedListNode.value);
  }

  // 스택의 연결 리스트의 string 표현을 리턴
  toString(callback) {
    return this.linkedList.toString(callback);
  }
}
```

\[참조\]

[Wikipedia](<https://en.wikipedia.org/wiki/Stack_(abstract_data_type)>)  
[YouTube](https://www.youtube.com/watch?v=wjI1WNcIntg&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=3&)
