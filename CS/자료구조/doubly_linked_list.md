[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/doubly-linked-list/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/doubly-linked-list/README.ko-KR.md)

## **Doubly Linked List (이중 연결 리스트)**

\- 순차적으로 링크된 노드라는 레코드 세트로 구성된 링크된 데이터 구조

\- 각 노드에는 **링크**라고 하는 두 개의 필드가 있으며, 노드 순서에서 **이전 노드와 다음 노드에 대한 참조**를 가진다

\- 시작 및 종료 노드의 이전 및 다음 링크는 각각 리스트의 순회를 용이하게 하기 위해 일종의 종결자(**센티넬코드 또는 null**)를 나타낸다

\- 센티넬 노드가 하나만 있으면, 목록이 센티넬 노드를 통해서 원형으로 연결된다

\- 동일한 데이터 항목으로 구성되어 있지만, 반대 순서로 두 개의 [단일 연결 리스트](https://sia-world.tistory.com/entry/Singly-Linked-List-%EB%8B%A8%EB%B0%A9%ED%96%A5-%EC%97%B0%EA%B2%B0-%EB%A6%AC%EC%8A%A4%ED%8A%B8-%EC%97%B0%EA%B2%B0-%EB%A6%AC%EC%8A%A4%ED%8A%B8)로 개념화 할 수 있다

![이중 연결 리스트](https://upload.wikimedia.org/wikipedia/commons/5/5e/Doubly-linked-list.svg)

\- 두 개의 노드 링크를 사용하면 어느 방향으로든 리스트를 순회할 수 있다

\- 이중 연결 리스트에서 노드를 삽입/삭제하려면, 단일 연결 리스트보다 더 많은 링크를 변경해야 하지만, 첫 번째 노드 이외의 노드인 경우 작업을 추적할 필요가 없으므로 작업이 더 단순해져 잠재적으로 더 효율적이다

\- 리스트 순회 중 이전 노드 또는 링크를 수정할 수 있도록 이전 노드를 찾기 위해 리스트를 순회할 필요가 없다

### 1) 복잡도

#### 시간복잡도

| **접근** | **탐색** | **삽입** | **삭제** |
| -------- | -------- | -------- | -------- |
| O(n)     | O(n)     | O(1)     | O(n)     |

#### 공간복잡도

O(n)

### 2) 자바스크립트로 구현해보기

\- head와 tail을 이용한 구현

```javascript
class Comparator {
  constructor(compareFunction) {
    this.compare = compareFunction || Comparator.defaultCompareFunction;
  }
  static defaultCompareFunction(a, b) {
    if (a === b) {
      return 0;
    }
    return a < b ? -1 : 1;
  }

  equal(a, b) {
    return this.compare(a, b) === 0;
  }

  lessThan(a, b) {
    return this.compare(a, b) < 0;
  }

  greaterThan(a, b) {
    return this.compare(a, b) > 0;
  }

  lessThanOrEqual(a, b) {
    return this.lessThan(a, b) || this.equal(a, b);
  }

  greaterThanOrEqual(a, b) {
    return this.greaterThan(a, b) || this.equal(a, b);
  }

  reverse() {
    const compareOriginal = this.compare;
    this.compare = (a, b) => compareOriginal(b, a);
  }
}

class DoublyLinkedListNode {
  constructor(value, next = null, previous = null) {
    this.value = value;
    this.next = next;
    this.previous = previous;
  }
  toString(callback) {
    return callback ? callback(this.value) : `${this.value}`;
  }
}

class DoublyLinkedList {
  constructor(comparatorFunction) {
    this.head = null;
    this.tail = null;
    this.compare = new Comparator(comparatorFunction);
  }

  // 맨 앞에 추가
  prepend(value) {
    // head 노드 만들기
    const newNode = new DoublyLinkedListNode(value, this.head);

    // 만약 head 있다면, 더이상 head 아니므로, previous를 새 노드(head)로 만든다
    // 그런 다음 새 노드를 head로 표시
    if (this.head) {
      this.head.previous = newNode;
    }
    this.head = newNode;
    // tail 없다면 새 노드로 tail 만들기
    if (!this.tail) {
      this.tail = newNode;
    }
    return this;
  }

  // 맨 뒤에 추가
  append(value) {
    const newNode = new DoublyLinkedListNode(value);
    // head 없으면 새 노드를 head로 만들기
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
      return this;
    }
    // 연결 리스트 끝에 새 노드 붙이기
    this.tail.next = newNode;
    // 새 노드 previous에 현재 tail 붙이기
    newNode.previous = this.tail;
    // tail이 될 새 노드 설정
    this.tail = newNode;
    return this;
  }

  // 값이 value인 노드 삭제
  delete(value) {
    if (!this.head) {
      return null;
    }
    let deletedNode = null;
    let currentNode = this.head;
    while (currentNode) {
      if (this.compare.equal(currentNode.value, value)) {
        deletedNode = currentNode;
        if (deletedNode === this.head) {
          // head를 삭제하려면
          // head를 새로운 head가 될 두번째 노드로 설정
          this.head = deletedNode.next;
          // 새 head previous를 null로 설정
          if (this.head) {
            this.head.previous = null;
          }
          // 만약 리스트의 모든 노드가 인자로 넘겨진 같은 값이라면,
          // 모든 노드가 삭제되므로 tail이 업데이트해야 한다
          if (deletedNode === this.tail) {
            this.tail = null;
          }
        } else if (deletedNode === this.tail) {
          // tail을 삭제하려면
          // Set tail to second last node, which will become new tail.
          // tail을 새로운 tail이 될 마지막에서 두번째 노드로 설정
          this.tail = deletedNode.previous;
          this.tail.next = null;
        } else {
          // 중간에 있는 노드를 삭제하려면
          const previousNode = deletedNode.previous;
          const nextNode = deletedNode.next;
          previousNode.next = nextNode;
          nextNode.previous = previousNode;
        }
      }
      currentNode = currentNode.next;
    }
    return deletedNode;
  }

  // 값이 value인 노드 찾기
  find({value = undefined, callback = undefined}) {
    if (!this.head) {
      return null;
    }
    let currentNode = this.head;
    while (currentNode) {
      // callback true라면 callback으로 찾기
      if (callback && callback(currentNode.value)) {
        return currentNode;
      }
      // value가 undefined가 아니라면 value로 비교하기
      if (value !== undefined && this.compare.equal(currentNode.value, value)) {
        return currentNode;
      }
      currentNode = currentNode.next;
    }
    return null;
  }

  // tail 삭제
  deleteTail() {
    if (!this.tail) {
      // No tail to delete.
      return null;
    }
    if (this.head === this.tail) {
      // 연결 리스트에 노드가 하나만 있다면
      const deletedTail = this.tail;
      this.head = null;
      this.tail = null;
      return deletedTail;
    }
    // 연결 리스트에 노드가 많다면
    const deletedTail = this.tail;
    this.tail = this.tail.previous;
    this.tail.next = null;
    return deletedTail;
  }

  // head 삭제
  deleteHead() {
    if (!this.head) {
      return null;
    }
    const deletedHead = this.head;
    if (this.head.next) {
      this.head = this.head.next;
      this.head.previous = null;
    } else {
      this.head = null;
      this.tail = null;
    }
    return deletedHead;
  }

  // Linked List -> Array
  toArray() {
    const nodes = [];
    let currentNode = this.head;
    while (currentNode) {
      nodes.push(currentNode);
      currentNode = currentNode.next;
    }
    return nodes;
  }

  // Array -> Linked List
  fromArray(values) {
    values.forEach((value) => this.append(value));
    return this;
  }

  // 현재 연결 리스트에 있는 노드의 모음
  toString(callback) {
    return this.toArray()
      .map((node) => node.toString(callback))
      .toString();
  }

  // 뒤집기
  reverse() {
    let currNode = this.head;
    let prevNode = null;
    let nextNode = null;
    while (currNode) {
      // 다음 노드 저장
      nextNode = currNode.next;
      prevNode = currNode.previous;
      // 이전 노드에 연결되도록 현재 노드를 다음 노드와 변경
      currNode.next = prevNode;
      currNode.previous = nextNode;
      // prevNode, currNode 1단계 움직이기
      prevNode = currNode;
      currNode = nextNode;
    }
    // head, tail 리셋
    this.tail = this.head;
    this.head = prevNode;
    return this;
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Doubly_linked_list)  
[YouTube](https://www.youtube.com/watch?v=JdQeNxWCguQ&t=7s&index=72&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
