[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/README.md)

## **Tree (트리)**

\- 연결된 노드 집합이 있는 계층적 트리 구조를 나타내는 널리 사용되는 추상 데이터타입(abstract data type, ADT)

\- 루트 노드에서 시작하는 노드들의 집합으로 재귀적으로 정의될 수 있다

\- 각 노드는 노드(자식)에 대한 참조 목록과 함께 값으로 구성된 데이터 구조

\- 각 노드는 많은 자식에 연결될 수 있지만, 부모가 없는 루트 노드를 제외하고는 정확히 하나의 부모에만 연결되어야 한다

![Tree](https://upload.wikimedia.org/wikipedia/commons/f/f7/Binary_tree.svg)

\- 7이라는 레이블이 지정된 노드에는 2와 6이라는 레이블이 지정된 두 개의 자식과 2라는 레이블이 지정된 한 개의 부모가 있다

### 1) 시간복잡도

#### 이진트리

\- O(logn)

빅오 표기법은 최대값을 기준으로 표기하기 때문에, 제일 말단 노드에 원하는 값이 있을경우 높이(h)만큼의 노드를 탐색해야한다

그럼 시간복잡도는 O(h)이지만 데이터의 수로 표기하는게 정석이기 때문에 이를 h를 n에 관한 식으로 바꿔서 표기해줘야한다

따라서

n=2^h−1

2^h=n−1

h=log(n−1)

이는 위에 그림처럼 루트 노드를 depth 0으로 지정한 경우 이와 같은 공식에 따라

O(h)=O(log(n−1))=>O(logn)

하지만 사람마다 케바케로 루트 노드를 depth 1로 지정하여 사용하는 경우가 있다

이 경우에는 아래와 같은 방식으로 시간 복잡도를 계산하면 된다

n=2^(h+1)−1

2^(h+1)=n+1

h+1=log(n+1)

h=log(n+1)−1

이와 같은 방식으로 계산되어 시간복잡도는

O(h)=O(log(n+1)−1)=>O(log(n))

따라서 어떻게 depth를 결정하던간에 시간 복잡도는

**O(log(n))**

### 2) 자바스크립트로 구현해보기

#### BinaryTreeNode (이진트리)

\- [해시 테이블](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/hash_table.md) 이용하기

```javascript
import HashTable from '../hash-table/HashTable';

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

export default class BinaryTreeNode {
  constructor(value = null) {
    this.left = null;
    this.right = null;
    this.parent = null;
    this.value = value;
    // 노드 관련 메타 정보가 여기에 저장될 수 있다
    this.meta = new HashTable();
    // 이 comparator는 이진 트리 노드를 서로 비교하는 데 사용됩니다.
    this.nodeComparator = new Comparator();
  }

  get leftHeight() {
    if (!this.left) {
      return 0;
    }
    return this.left.height + 1;
  }

  get rightHeight() {
    if (!this.right) {
      return 0;
    }
    return this.right.height + 1;
  }

  get height() {
    return Math.max(this.leftHeight, this.rightHeight);
  }

  get balanceFactor() {
    return this.leftHeight - this.rightHeight;
  }

  // 부모의 형제(존재하는 경우)를 가져온다
  get uncle() {
    // Check if current node has parent.
    if (!this.parent) {
      return undefined;
    }
    // 현재 노드의 부모의 부모 노드가 있는지 확인
    if (!this.parent.parent) {
      return undefined;
    }
    // 부모의 부모노드가 자식이 둘인지 확인
    if (!this.parent.parent.left || !this.parent.parent.right) {
      return undefined;
    }
    // 현재 노드에는 부모의 부모노드가 있고, 이 부모의 부모노드는 두 자식노드가 있다
    // 누가 삼촌(또 다른 자식)인지 알아보자
    if (this.nodeComparator.equal(this.parent, this.parent.parent.left)) {
      // 오른쪽이 삼촌
      return this.parent.parent.right;
    }
    // 왼쪽이 삼촌
    return this.parent.parent.left;
  }

  setValue(value) {
    this.value = value;
    return this;
  }

  setLeft(node) {
    // 왼쪽 노드가 분리될 예정이므로 왼쪽 노드의 부모를 재설정
    if (this.left) {
      this.left.parent = null;
    }
    // 왼쪽에 새 노드 붙이기
    this.left = node;
    // 현재 노드를 새 왼쪽 노드의 부모 노드로 만들기
    if (this.left) {
      this.left.parent = this;
    }
    return this;
  }

  setRight(node) {
    // 오른쪽 노드가 분리될 예정이므로 오른쪽 노드의 부모를 재설정
    if (this.right) {
      this.right.parent = null;
    }
    // 오른쪽에 새 노드 붙이기
    this.right = node;
    // 현재 노드를 새 오른쪽 노드의 부모 노드로 만들기
    if (node) {
      this.right.parent = this;
    }
    return this;
  }

  removeChild(nodeToRemove) {
    if (this.left && this.nodeComparator.equal(this.left, nodeToRemove)) {
      this.left = null;
      return true;
    }
    if (this.right && this.nodeComparator.equal(this.right, nodeToRemove)) {
      this.right = null;
      return true;
    }
    return false;
  }

  replaceChild(nodeToReplace, replacementNode) {
    if (!nodeToReplace || !replacementNode) {
      return false;
    }
    if (this.left && this.nodeComparator.equal(this.left, nodeToReplace)) {
      this.left = replacementNode;
      return true;
    }
    if (this.right && this.nodeComparator.equal(this.right, nodeToReplace)) {
      this.right = replacementNode;
      return true;
    }
    return false;
  }

  static copyNode(sourceNode, targetNode) {
    targetNode.setValue(sourceNode.value);
    targetNode.setLeft(sourceNode.left);
    targetNode.setRight(sourceNode.right);
  }

  traverseInOrder() {
    let traverse = [];
    // 왼쪽 노드 추가
    if (this.left) {
      traverse = traverse.concat(this.left.traverseInOrder());
    }
    // 루트 추가
    traverse.push(this.value);
    // 오른쪽 노드 추가
    if (this.right) {
      traverse = traverse.concat(this.right.traverseInOrder());
    }
    return traverse;
  }

  toString() {
    return this.traverseInOrder().toString();
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Linked_list)  
[YouTube](https://www.youtube.com/watch?v=njTh_OwMljA&index=2&t=1s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
