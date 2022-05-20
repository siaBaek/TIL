[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/queue/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/queue/README.ko-KR.md)

## **Queue (큐)**

\- 일종의 추상 데이터 타입이자 컬렉션

\- 큐 내부 **엔티티(Entity, 개체)**들은 순서를 유지한다

\- **인큐(Enqueue)**: 컬렉션의 가장 뒷 부분에 엔티티를 추가

\- **디큐(Dequeue):** 컬렉션의 가장 앞에 위치한 엔티티를 제거

\- 이것은 큐를 **선입선출** 자료 구조로 만든다. (추가된 첫 번째 요소가 가장 먼저 제거)

\- 이는 새로운 요소가 추가되면 이전에 추가되었던 모든 요소들을 제거해야 새로운 요소를 제거할 수 있다는것과 같은 의미

\- 또한 큐의 가장 앞에 위치한 요소를 반환하기 위한 작업이 입력되면 디큐 작업 없이 해당 요소를 반환한다.

\- 큐는 선형 자료 구조의 예시이며, 더 추상적으로는 순차적인 컬렉션이다.

\- 선입선출 자료 구조인 큐를 나타내면 다음과 같다.

![Queue](https://upload.wikimedia.org/wikipedia/commons/5/52/Data_Queue.svg)

### 1) 자바스크립트로 구현해보기

\- [Linked List](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/singly_linked_list.md) 이용하기

```javascript
import LinkedList from '../linked-list/LinkedList'; // 연결 리스트 import

export default class Queue {
  constructor() {
    // 연결 리스트와 구조가 유사하기 때문에 연결 리스트 기반으로 큐를 구현해보자
    // 둘 다 시작과 끝에 있는 요소에서 주로 작동한다
    // 큐의 enqueue/dequeue 작업은 연결 리스트의 append/deleteHead 와 비교된다
    this.linkedList = new LinkedList();
  }

  // 빈 큐인지 확인
  isEmpty() {
    return !this.linkedList.head;
  }

  // head value, 비어있다면 null
  peek() {
    if (this.isEmpty()) {
      return null;
    }
    return this.linkedList.head.value;
  }

  // enqueue(가장 뒤에 value 추가)
  enqueue(value) {
    this.linkedList.append(value);
  }

  // dequeue(head 삭제)
  dequeue() {
    const removedHead = this.linkedList.deleteHead();
    return removedHead ? removedHead.value : null;
  }

  // 큐의 연결 리스트의 string 표현을 리턴
  toString(callback) {
    return this.linkedList.toString(callback);
  }
}
```

\[참조\]

[Wikipedia](<https://en.wikipedia.org/wiki/Queue_(abstract_data_type)>)  
[YouTube](https://www.youtube.com/watch?v=wjI1WNcIntg&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=3&)
