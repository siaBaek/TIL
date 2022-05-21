[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/heap/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/heap/README.ko-KR.md)

## **Heap (힙)**

\- 아래에 설명된 힙 속성을 만족하는 전문화된 트리 기반 데이터구조

#### 최소 힙

- **최소 힙**에서 `P`가 `C`의 상위 노드라면 `P`의 키(값)는 `C`의 키보다 작거나 같습니다.
- 각 노드의 키(Key)값이 (자식 노드가 있다면) 그 자식의 키(Key)값보다 크지 않은(=작거나 같은) 트리
- 최소 트리이면서 완전 이진 트리이다 (\*완전 이진 트리: 노드를 삽입할 때, 왼쪽부터 차례대로 삽입하는 트리)

![MinHeap](https://upload.wikimedia.org/wikipedia/commons/6/69/Min-heap.png)

#### 최대 힙

- **최대 힙**에서 `P`의 키는 `C`의 키보다 크거나 같습니다.
- 각 노드의 키(Key)값이 (자식 노드가 있다면) 그 자식의 키(Key)값보다 작지 않은(=크거나 같은) 트리
- 최대 트리이면서 완전 이진 트리이다 (\*완전 이진 트리: 노드를 삽입할 때, 왼쪽부터 차례대로 삽입하는 트리)

![Heap](https://upload.wikimedia.org/wikipedia/commons/3/38/Max-Heap.svg)

- 상위 노드가 없는 힙의 상단에 있는 노드를 **루트 노드**라고 합니다.

### 1) 자바스크립트로 구현해보기

\- **힙** 구조 만들기

\- 힙을 기반으로 **최소 힙**과 **최대 힙** 만들기

#### 힙

\- Comparator

```javascript
export default class Comparator {
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
```

\- Heap

```javascript
import Comparator from '../../utils/comparator/Comparator';

export default class Heap {
  constructor(comparatorFunction) {
    if (new.target === Heap) {
      throw new TypeError('Cannot construct Heap instance directly');
    }
    // 힙의 배열 표현
    this.heapContainer = [];
    this.compare = new Comparator(comparatorFunction);
  }

  getLeftChildIndex(parentIndex) {
    return 2 * parentIndex + 1;
  }

  getRightChildIndex(parentIndex) {
    return 2 * parentIndex + 2;
  }

  getParentIndex(childIndex) {
    return Math.floor((childIndex - 1) / 2);
  }

  hasParent(childIndex) {
    return this.getParentIndex(childIndex) >= 0;
  }

  hasLeftChild(parentIndex) {
    return this.getLeftChildIndex(parentIndex) < this.heapContainer.length;
  }

  hasRightChild(parentIndex) {
    return this.getRightChildIndex(parentIndex) < this.heapContainer.length;
  }

  leftChild(parentIndex) {
    return this.heapContainer[this.getLeftChildIndex(parentIndex)];
  }

  rightChild(parentIndex) {
    return this.heapContainer[this.getRightChildIndex(parentIndex)];
  }

  parent(childIndex) {
    return this.heapContainer[this.getParentIndex(childIndex)];
  }

  swap(indexOne, indexTwo) {
    const tmp = this.heapContainer[indexTwo];
    this.heapContainer[indexTwo] = this.heapContainer[indexOne];
    this.heapContainer[indexOne] = tmp;
  }

  peek() {
    if (this.heapContainer.length === 0) {
      return null;
    }
    return this.heapContainer[0];
  }

  poll() {
    if (this.heapContainer.length === 0) {
      return null;
    }
    if (this.heapContainer.length === 1) {
      return this.heapContainer.pop();
    }
    const item = this.heapContainer[0];
    // 마지막 요소를 끝에서 헤드로 이동
    this.heapContainer[0] = this.heapContainer.pop();
    this.heapifyDown();
    return item;
  }

  add(item) {
    this.heapContainer.push(item);
    this.heapifyUp();
    return this;
  }

  remove(item, comparator = this.compare) {
    // 제거할 항목 수를 찾는다
    const numberOfItemsToRemove = this.find(item, comparator).length;
    for (let iteration = 0; iteration < numberOfItemsToRemove; iteration += 1) {
      // 각 heapify 프로세스 후에 인덱스가 변경되기 때문에 제거 후 매번 제거할 아이템 인덱스를 찾아야 한다
      const indexToRemove = this.find(item, comparator).pop();
      // 힙에서 마지막 하위 항목을 제거해야 할 경우 제거하기만 하면 된다
      // 나중에 힙을 heapify할 필요는 없다
      if (indexToRemove === this.heapContainer.length - 1) {
        this.heapContainer.pop();
      } else {
        // 힙의 마지막 요소를 빈 (제거된)위치로 이동
        this.heapContainer[indexToRemove] = this.heapContainer.pop();
        // parent 얻기
        const parentItem = this.parent(indexToRemove);
        // 만약 삭제할 노드에 부모 또는 올바른 순서의 부모가 없다면 heapifyDown하고
        // 그렇지 않다면 heapifyUp
        if (
          this.hasLeftChild(indexToRemove) &&
          (!parentItem || this.pairIsInCorrectOrder(parentItem, this.heapContainer[indexToRemove]))
        ) {
          this.heapifyDown(indexToRemove);
        } else {
          this.heapifyUp(indexToRemove);
        }
      }
    }
    return this;
  }

  find(item, comparator = this.compare) {
    const foundItemIndices = [];
    for (let itemIndex = 0; itemIndex < this.heapContainer.length; itemIndex += 1) {
      if (comparator.equal(item, this.heapContainer[itemIndex])) {
        foundItemIndices.push(itemIndex);
      }
    }
    return foundItemIndices;
  }

  isEmpty() {
    return !this.heapContainer.length;
  }

  toString() {
    return this.heapContainer.toString();
  }

  heapifyUp(customStartIndex) {
    // 힙 컨테이너의 마지막 요소(배열의 마지막 또는 트리의 왼쪽아래)를 가져와서
    // 상위 요소 기준으로 올바른 순서가 될 때까지 lift up
    let currentIndex = customStartIndex || this.heapContainer.length - 1;
    while (
      this.hasParent(currentIndex) &&
      !this.pairIsInCorrectOrder(this.parent(currentIndex), this.heapContainer[currentIndex])
    ) {
      this.swap(currentIndex, this.getParentIndex(currentIndex));
      currentIndex = this.getParentIndex(currentIndex);
    }
  }

  heapifyDown(customStartIndex = 0) {
    // 부모 요소를 해당 자식 요소와 비교하고,
    // 부모를 적절한 자식(MinHeap의 경우 가장 작은 자식, MaxHeap의 경우 가장 큰 자식)로 바꾼다
    // 바꾼 후 다음 자식도 같은 방식으로 수행
    let currentIndex = customStartIndex;
    let nextIndex = null;
    while (this.hasLeftChild(currentIndex)) {
      if (
        this.hasRightChild(currentIndex) &&
        this.pairIsInCorrectOrder(this.rightChild(currentIndex), this.leftChild(currentIndex))
      ) {
        nextIndex = this.getRightChildIndex(currentIndex);
      } else {
        nextIndex = this.getLeftChildIndex(currentIndex);
      }
      if (
        this.pairIsInCorrectOrder(this.heapContainer[currentIndex], this.heapContainer[nextIndex])
      ) {
        break;
      }
      this.swap(currentIndex, nextIndex);
      currentIndex = nextIndex;
    }
  }

  /**
   * 힙 요소 쌍이 올바른 순서인지 확인
   * MinHeap의 경우 첫 번째 요소는 항상 더 작거나 같아야 한다
   * MaxHeap의 경우 첫 번째 요소는 항상 더 크거나 같아야 한다
   */
  /* istanbul ignore next */
  pairIsInCorrectOrder(firstElement, secondElement) {
    throw new Error(`
      You have to implement heap pair comparision method
      for ${firstElement} and ${secondElement} values.
    `);
  }
}
```

#### 최소 힙과 최대 힙

\- 최소 힙

```javascript
import Heap from './Heap';

export default class MinHeap extends Heap {
  pairIsInCorrectOrder(firstElement, secondElement) {
    return this.compare.lessThanOrEqual(firstElement, secondElement); // boolean
  }
}
```

\- 최대 힙

```javascript
import Heap from './Heap';

export default class MaxHeap extends Heap {
  pairIsInCorrectOrder(firstElement, secondElement) {
    return this.compare.greaterThanOrEqual(firstElement, secondElement); // boolean
  }
}
```

\[참조\]

[Wikipedia](<https://en.wikipedia.org/wiki/Heap_(data_structure)>)  
[YouTube](https://www.youtube.com/watch?v=t0Cq6tVNRBA&index=5&t=0s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
