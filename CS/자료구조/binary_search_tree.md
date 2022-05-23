[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/binary-search-tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/binary-search-tree/README.md)

## **Binary Search Tree (BST, 이진 탐색 트리)**

다음과 같은 속성이 있는 이진 트리 자료 구조라고 할 수 있다

- 각 노드에 값이 있다
- 값들은 전순서가 있다
- 노드의 왼쪽 서브트리에는 그 노드의 값보다 작은 값들을 지닌 노드들로 이루어져 있다
- 노드의 오른쪽 서브트리에는 그 노드의 값보다 큰 값들을 지닌 노드들로 이루어져 있다
- 좌우 하위 트리는 각각이 다시 이진 탐색 트리여야 한다

\- ordered binary tree 또는 sorted binary tree라고 부르기도 한다

\- 특정 유형의 컨테이너 즉, 메모리에 **아이템**(숫자, 이름 등)을 저장하는 데이터 구조이다

\- 빠른 검색, 삽입, 삭제가 가능하다

\- 아이템의 동적 셋 또는 키로 항목을 찾을 수 있는 룩업 테이블(예: 이름으로 사용자의 전화 번호 찾기)을 구현하는데 사용할 수 있다

\- 키를 정렬된 순서대로 유지하므로 룩업 및 기타 작업이 이진 검색의 원리를 사용할 수 있다.  
\- 트리(또는 새 키를 삽입할 장소)에서 키를 찾을 때, 트리를 루트 노드부터 말단 노드까지 가로지르며, 트리의 노드에 저장된 키와 비교하고, 그 비교에 기초하여 왼쪽 또는 오른쪽 하위 트리에서 계속 검색하기로 결정한다

\- 평균적으로, 이것은 각 비교가 트리의 약 절반을 건너뛸 수 있다는 것을 의미하며, 따라서 각 검색, 삽입 또는 삭제는 트리에 저장된 아이템 수의 로그에 비례하는 시간이 소요된다  
\- 이것은 (정렬되지 않은) 배열에서 키별로 아이템을 찾는 데 필요한 선형 시간보다 훨씬 빠르지만 해시 테이블에 대한 해당 작업보다 느리다

![Binary Search Tree](https://upload.wikimedia.org/wikipedia/commons/d/da/Binary_search_tree.svg)

\- 크기가 9이고 깊이가 3이고 루트가 8인 이진 검색 트리 (말단 노드가 그려지지 않았다)

### 1) 복잡도

#### 시간복잡도

|  Access   |  Search   | Insertion | Deletion  |
| :-------: | :-------: | :-------: | :-------: |
| O(log(n)) | O(log(n)) | O(log(n)) | O(log(n)) |

#### 공간복잡도

O(n)

### 2) 이진 탐색 트리에서의 검색/삽입/삭제

#### 검색

- 이진탐색트리에서 키 x를 가진 노드를 검색하고자 할때, 트리에 해당 노드가 존재하면 해당 노드를 리턴하고, 존재하지 않으면 NULL을 리턴한다
- 검색하고자 하는 값을 루트노드와 먼저 비교하고, 일치할 경우 루트노드를 리턴한다
  - 불일치하고 검색하고자 하는 값이 루트노드의 값보다 작을 경우 왼쪽 서브트리에서 재귀적으로 검색한다
  - 불일치하고 검색하고자 하는 값이 루트노드의 값과 같거나 큰 경우 오른쪽 서브트리에서 재귀적으로 검색한다

#### 삽입

- 삽입을 하기 전, 검색을 수행한다
- 트리를 검색한 후 키와 일치하는 노드가 없으면 마지막 노드에서 키와 노드의 크기를 비교하여서 왼쪽이나 오른쪽에 새로운 노드를 삽입한다

#### 삭제

삭제하려는 노드의 자식 수에 따라

- 자식노드가 없는 노드(리프 노드) 삭제 : 해당 노드를 단순히 삭제한다
- 자식노드가 1개인 노드 삭제 : 해당 노드를 삭제하고 그 위치에 해당 노드의 자식노드를 대입한다
- 자식노드가 2개인 노드 삭제 : 삭제하고자 하는 노드의 값을 해당 노드의 왼쪽 서브트리에서 가장 큰값으로 변경하거나, 오른쪽 서브트리에서 가장 작은 값으로 변경한 뒤, 해당 노드(왼쪽서브트리에서 가장 큰 값을 가지는 노드 또는 오른쪽 서브트리에서 가장 작은 값을 가지는 노드)를 삭제한다

### 3) 자바스크립트로 구현해보기

\- [binary tree node](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/tree.md) 이용하기

```javascript
// Binary Search Tree Node
import BinaryTreeNode from '../BinaryTreeNode';
import Comparator from '../../../utils/comparator/Comparator';

export default class BinarySearchTreeNode extends BinaryTreeNode {
  constructor(value = null, compareFunction = undefined) {
    super(value);
    // 이 Comparator는 노드 값을 서로 비교하는 데 사용된다
    this.compareFunction = compareFunction;
    this.nodeValueComparator = new Comparator(compareFunction);
  }

  insert(value) {
    if (this.nodeValueComparator.equal(this.value, null)) {
      this.value = value;
      return this;
    }
    if (this.nodeValueComparator.lessThan(value, this.value)) {
      // 왼쪽에 추가
      if (this.left) {
        return this.left.insert(value);
      }
      const newNode = new BinarySearchTreeNode(value, this.compareFunction);
      this.setLeft(newNode);
      return newNode;
    }

    if (this.nodeValueComparator.greaterThan(value, this.value)) {
      // 오른쪽에 추가
      if (this.right) {
        return this.right.insert(value);
      }
      const newNode = new BinarySearchTreeNode(value, this.compareFunction);
      this.setRight(newNode);
      return newNode;
    }
    return this;
  }

  find(value) {
    // 루트 확인
    if (this.nodeValueComparator.equal(this.value, value)) {
      return this;
    }
    if (this.nodeValueComparator.lessThan(value, this.value) && this.left) {
      // 왼쪽 노드 확인
      return this.left.find(value);
    }
    if (this.nodeValueComparator.greaterThan(value, this.value) && this.right) {
      // 오른쪽 노드 확인
      return this.right.find(value);
    }
    return null;
  }

  contains(value) {
    return !!this.find(value);
  }

  remove(value) {
    const nodeToRemove = this.find(value);
    if (!nodeToRemove) {
      throw new Error('Item not found in the tree');
    }
    const {parent} = nodeToRemove;
    if (!nodeToRemove.left && !nodeToRemove.right) {
      // 말단 노드이므로 자식이 없다
      if (parent) {
        // 노드에 부모가 있다면, 부모 노드에서 이 노드에 대한 포인터를 제거하기만 하면 된다
        parent.removeChild(nodeToRemove);
      } else {
        // 노드에 부모가 없다면, 현재 노드 값만 지우면 된다
        nodeToRemove.setValue(undefined);
      }
    } else if (nodeToRemove.left && nodeToRemove.right) {
      // 노드에 두 자식 노드가 있다면
      // 다음으로 큰 값(오른쪽 가지의 최소값)을 찾고 현재 값 노드를 다음으로 큰 값으로 바꿉니다.
      const nextBiggerNode = nodeToRemove.right.findMin();
      if (!this.nodeComparator.equal(nextBiggerNode, nodeToRemove.right)) {
        this.remove(nextBiggerNode.value);
        nodeToRemove.setValue(nextBiggerNode.value);
      } else {
        // 다음 오른쪽 값이 큰 값이고 왼쪽 자식 값이 없는 경우 삭제할 노드를 오른쪽 노드로 대체
        nodeToRemove.setValue(nodeToRemove.right.value);
        nodeToRemove.setRight(nodeToRemove.right.right);
      }
    } else {
      // 노드에 자식 노드가 하나만 있다면
      // 이 자식 노드를 현재 노드의 부모 노드의 직접적인 자식 노드로 만든다
      const childNode = nodeToRemove.left || nodeToRemove.right;
      if (parent) {
        parent.replaceChild(nodeToRemove, childNode);
      } else {
        BinaryTreeNode.copyNode(childNode, nodeToRemove);
      }
    }
    // 제거된 노드의 부모 노드를 지웁니다.
    nodeToRemove.parent = null;
    return true;
  }

  findMin() {
    if (!this.left) {
      return this;
    }
    return this.left.findMin();
  }
}
```

```javascript
// Binary Search Tree
import BinarySearchTreeNode from './BinarySearchTreeNode';

export default class BinarySearchTree {
  constructor(nodeValueCompareFunction) {
    this.root = new BinarySearchTreeNode(null, nodeValueCompareFunction);
    // nodeComparator 루트에서 덮어쓰기
    this.nodeComparator = this.root.nodeComparator;
  }

  insert(value) {
    return this.root.insert(value);
  }

  contains(value) {
    return this.root.contains(value);
  }

  remove(value) {
    return this.root.remove(value);
  }

  toString() {
    return this.root.toString();
  }
}
```

\[참조\]

[Wikipedia1](https://en.wikipedia.org/wiki/Binary_search_tree)  
[Wikipedia2](https://ko.wikipedia.org/wiki/%EC%9D%B4%EC%A7%84_%ED%83%90%EC%83%89_%ED%8A%B8%EB%A6%AC)  
[YouTube](https://www.youtube.com/watch?v=wcIRPqTR3Kc&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=9&t=0s)  
[BST Interactive Visualisations](https://www.cs.usfca.edu/~galles/visualization/BST.html)
