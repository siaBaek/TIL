[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/linked-list/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/linked-list/README.ko-KR.md)

## **Singly Linked List (단일 연결 리스트)**

\- 데이터 요소의 **선형** 집합

\- 이 집합에서 논리적 저장 순서는 메모리의 **물리적 저장 순서와 일치하지 않는다**

\- 순서를 표현하는 **노드**들의 집합으로 이루어져 있다

\- 각각의 노드들은 **데이터**와 다음 노드를 가리키는 **레퍼런스(링크)**로 이루어져 있다

\- 링크는 다음 데이터를 가리키며 링크를 통해 데이터를 추가/삭제/탐색 등을 할 수 있다

\- 순회하는 동안 **순서에 상관없이 효율적인 삽입/삭제**가 가능하다

\- 링크 하나에 4 byte 메모리를 차지하며, 더블 링크드일 경우엔 노드 하나에 8 byte의 링크 메모리를 차지한다.

![링크드 리스트](https://upload.wikimedia.org/wikipedia/commons/6/6d/Singly-linked-list.svg)

### 1) 배열 리스트와 연결 리스트 비교

| **배열 리스트** (가장 간단한 메모리 데이터 구조로 사용이 쉬우며 데이터 참조가 쉽고, 동일한 데이터 타입을 연속적으로 저장할 수 있다)                                 |                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **장점** \- 데이터 **탐색/정렬**이 용이하다 (인덱스를 통한 빠른 접근이 가능하다) \- 연결 리스트에 비해 배열이 더 나은 **캐시 지역성(cache locality)**을 가지고 있다 | **단점** \- 데이터 삽입/삭제가 복잡하다 (JS를 제외한 C, Java 등의 언어에서 배열은 고정된 크기를 가지고 있어서 배열의 처음이나 중간에서 데이터 삽입/삭제시 비싼 연산을 빈번히 해야 한다)                   |
| **연결 리스트** (배열의 단점을 극복)                                                                                                                                |                                                                                                                                                                                                           |
| **장점** \- 데이터 **삽입/삭제**가 용이하다                                                                                                                         | **단점** \- 데이터 탐색/정렬이 복잡하다 (특정 위치 데이터를 검색하기 위해서는 처음부터 끝까지 순회해야 해서 탐색에 비효율적이다) \- 접근 시간이 선형이며 병렬 처리가 불가능하다 \- 차지하는 메모리가 크다 |

∴ 데이터의 탐색과 정렬을 자주 한다면 배열 리스트, 데이터의 삽입과 삭제가 많다면 연결 리스트

### 2) 복잡도

#### 시간복잡도

| **접근** | **탐색** | **삽입** | **삭제** |
| -------- | -------- | -------- | -------- |
| O(n)     | O(n)     | O(1)     | O(n)     |

#### 공간복잡도

O(n)

### 3) 자바스크립트로 구현해보기

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

class LinkedListNode {
  constructor(value, next = null) {
    this.value = value;
    this.next = next;
  }

  toString(callback) {
    return callback ? callback(this.value) : `${this.value}`;
  }
}

class LinkedList {
  constructor(comparatorFunction) {
    this.head = null;
    this.tail = null;
    this.compare = new Comparator(comparatorFunction);
  }

  // 맨 앞에 추가
  prepend(value) {
    // head 노드 만들기
    const newNode = new LinkedListNode(value, this.head);
    this.head = newNode;
    // tail 없다면 tail 노드 만들기
    if (!this.tail) {
      this.tail = newNode;
    }
    return this;
  }

  // 맨 뒤에 추가
  append(value) {
    const newNode = new LinkedListNode(value);
    // head 아직 없다면 head 노드 만들기
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
      return this;
    }
    // 연결 리스트 끝부분에 노드 붙이기
    this.tail.next = newNode;
    this.tail = newNode;
    return this;
  }

  // 추가하고 싶은 부분에 추가
  insert(value, rawIndex) {
    const index = rawIndex < 0 ? 0 : rawIndex;
    if (index === 0) {
      this.prepend(value);
    } else {
      let count = 1;
      let currentNode = this.head;
      const newNode = new LinkedListNode(value);
      while (currentNode) {
        if (count === index) break;
        currentNode = currentNode.next;
        count += 1;
      }
      if (currentNode) {
        newNode.next = currentNode.next;
        currentNode.next = newNode;
      } else {
        if (this.tail) {
          this.tail.next = newNode;
          this.tail = newNode;
        } else {
          this.head = newNode;
          this.tail = newNode;
        }
      }
    }
    return this;
  }

  // value를 element로 갖는 노드 삭제
  delete(value) {
    if (!this.head) {
      return null;
    }
    let deletedNode = null;
    // head 삭제해야하는 경우, head와 다른 다음 노드를 만들어서 새 head로 만들기
    while (this.head && this.compare.equal(this.head.value, value)) {
      deletedNode = this.head;
      this.head = this.head.next;
    }
    let currentNode = this.head;
    if (currentNode !== null) {
      // 다음 노드를 삭제해야 하는 경우, 다음 노드를 다다음 노드로 만듭니다.
      while (currentNode.next) {
        if (this.compare.equal(currentNode.next.value, value)) {
          deletedNode = currentNode.next;
          currentNode.next = currentNode.next.next;
        } else {
          currentNode = currentNode.next;
        }
      }
    }
    // tail 삭제해야하는지 확인
    if (this.compare.equal(this.tail.value, value)) {
      this.tail = currentNode;
    }
    return deletedNode;
  }

  // value를 element로 갖는 노드 찾기
  find({value = undefined, callback = undefined}) {
    if (!this.head) {
      return null;
    }
    let currentNode = this.head;
    while (currentNode) {
      // callback이 true면, 콜백으로 노드 찾기
      if (callback && callback(currentNode.value)) {
        return currentNode;
      }
      // value가 undefined 아니라면 value로 비교
      if (value !== undefined && this.compare.equal(currentNode.value, value)) {
        return currentNode;
      }
      currentNode = currentNode.next;
    }
    return null;
  }

  // 맨 뒤 노드 삭제
  deleteTail() {
    const deletedTail = this.tail;
    if (this.head === this.tail) {
      // 연결 리스트에 하나의 노드만 있다면
      this.head = null;
      this.tail = null;
      return deletedTail;
    }
    // 연결 리스트에 노드가 많다면
    // 마지막 노드로 돌아가서, 마지막 이전 노드에 대해 다음 링크를 삭제
    let currentNode = this.head;
    while (currentNode.next) {
      if (!currentNode.next.next) {
        currentNode.next = null;
      } else {
        currentNode = currentNode.next;
      }
    }
    this.tail = currentNode;
    return deletedTail;
  }

  // 맨 앞 노드 삭제
  deleteHead() {
    if (!this.head) {
      return null;
    }
    const deletedHead = this.head;
    if (this.head.next) {
      this.head = this.head.next;
    } else {
      this.head = null;
      this.tail = null;
    }
    return deletedHead;
  }

  // Array -> Linked List
  fromArray(values) {
    values.forEach((value) => this.append(value));
    return this;
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
      // 이전 노드에 연결되도록 현재 노드를 다음 노드와 변경
      currNode.next = prevNode;
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

[Wikipedia](https://en.wikipedia.org/wiki/Linked_list)  
[YouTube](https://www.youtube.com/watch?v=njTh_OwMljA&index=2&t=1s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
