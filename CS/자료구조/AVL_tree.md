[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/avl-tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/avl-tree/README.md)

## **AVL Tree (AVL 트리)**

\- 발명자 이름인 **A**delson-**V**elsky and **L**andis에서 따온 이름

\- 자가 균형 이진 탐색 트리

\- 자가 균형 데이터 구조 중 처음으로 발명되었다

\- 어떤 노드든 두 자식 서브트리의 높이는 최대 1만큼 차이가 난다

\- 만약 어떤 시점에서 높이 차이가 1보다 커지면, 이 속성을 유지하기 위하여 스스로 균형을 잡는다

\- 검색, 삽입, 삭제는 모두 평균과 최악의 경우 O(logn)의 시간복잡도가 걸린다 (n은 작업 전 트리의 노드 수)

\- 삽입과 삭제는 한 번 이상의 트리 회전을 통해 트리의 균형을 재조정해야 할 수 있다

![AVL Tree](https://upload.wikimedia.org/wikipedia/commons/f/fd/AVL_Tree_Example.gif)

🔼 AVL 트리에 여러 요소를 삽입하는 모습을 보여주는 애니메이션 (왼, 오, 왼-오, 오-왼 회전)

![AVL Tree](https://upload.wikimedia.org/wikipedia/commons/a/ad/AVL-tree-wBalance_K.svg)

🔼 균형 요인이 있는 AVL 트리(녹색)

### 1) AVL Tree Rotations

#### **Left-Left Rotation**

[##_Image|kage@u8IvD/btrC0M4Xpqv/u4Y2AEvvr66LS0YCjhdC1k/img.png|CDM|1.3|{"originWidth":1024,"originHeight":400,"style":"alignCenter"}_##]

#### **Right-Right Rotation**

[##_Image|kage@dKIFkw/btrC2wmxVhW/JrqxcG3LZeEpPQk4DdQUcK/img.png|CDM|1.3|{"originWidth":1024,"originHeight":400,"style":"alignCenter"}_##]

#### **Left-Right Rotation**

[##_Image|kage@cak8UK/btrCXxUICZQ/arOyZHYgf8dA4w7q9ZADlK/img.png|CDM|1.3|{"originWidth":1024,"originHeight":400,"style":"alignCenter"}_##]

#### **Right-Left Rotation**

[##_Image|kage@IZrLs/btrC0N3TYq4/CklolBck545r5Y3fz9EtE1/img.png|CDM|1.3|{"originWidth":1024,"originHeight":400,"style":"alignCenter"}_##]

### 2) 복잡도

#### 시간복잡도

| **Space** | **Search** | **Insert** | **Delete** |
| --------- | ---------- | ---------- | ---------- |
| O(n)      | O(logn)    | O(logn)    | O(logn)    |

### 3) 자바스크립트로 구현해보기

\- [이진탐색트리](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/binary_search_tree.md) 이용하기

```javascript
import BinarySearchTree from '../binary-search-tree/BinarySearchTree';

export default class AvlTree extends BinarySearchTree {
  insert(value) {
    // 일반 이진탐색트리 삽입
    super.insert(value);
    // 루트로 올라가며 균형 요인을 살펴보자
    let currentNode = this.root.find(value);
    while (currentNode) {
      this.balance(currentNode);
      currentNode = currentNode.parent;
    }
  }

  remove(value) {
    // 표준 이진탐색트리 삭제
    super.remove(value);
    // 루트 노드에서 시작하여 트리 균형을 조정
    this.balance(this.root);
  }

  balance(node) {
    // 균형 요인이 올바르지 않으면 노드의 균형을 조정
    if (node.balanceFactor > 1) {
      // 왼쪽 회전
      if (node.left.balanceFactor > 0) {
        // 왼쪽-왼쪽 회전
        this.rotateLeftLeft(node);
      } else if (node.left.balanceFactor < 0) {
        // 왼쪽-오른쪽 회전
        this.rotateLeftRight(node);
      }
    } else if (node.balanceFactor < -1) {
      // 오른쪽 회전
      if (node.right.balanceFactor < 0) {
        // 오른쪽-오른쪽 회전
        this.rotateRightRight(node);
      } else if (node.right.balanceFactor > 0) {
        // 오른쪽-왼쪽 회전
        this.rotateRightLeft(node);
      }
    }
  }

  rotateLeftLeft(rootNode) {
    // 루트 노드(rootNode)에서 왼쪽 노드(leftNode)를 분리
    const leftNode = rootNode.left;
    rootNode.setLeft(null);
    // leftNode를 rootNode 부모의 자식 노드로 만든다
    if (rootNode.parent) {
      rootNode.parent.setLeft(leftNode);
    } else if (rootNode === this.root) {
      // 루트 노드가 루트인 경우, leftNode를 새 루트로 만든다
      this.root = leftNode;
    }
    // leftNode에 오른쪽 자식이 있는 경우 이를 분리하여 rootNode의 왼쪽 자식으로 연결
    if (leftNode.right) {
      rootNode.setLeft(leftNode.right);
    }
    // rootNode를 leftNode의 오른쪽에 연결
    leftNode.setRight(rootNode);
  }

  rotateLeftRight(rootNode) {
    // 왼쪽 노드가 교체될 예정이므로 rootNode에서 노드를 분리
    const leftNode = rootNode.left;
    rootNode.setLeft(null);
    // leftNode에서 오른쪽 노드 분리
    const leftRightNode = leftNode.right;
    leftNode.setRight(null);
    // leftRightNode의 왼쪽 하위 트리를 유지
    if (leftRightNode.left) {
      leftNode.setRight(leftRightNode.left);
      leftRightNode.setLeft(null);
    }
    // rootNode에 leftRightNode 붙이기
    rootNode.setLeft(leftRightNode);
    // leftNode를 leftRightNode의 왼쪽 노드로 붙이기
    leftRightNode.setLeft(leftNode);
    // 왼쪽-왼쪽 회전 실행
    this.rotateLeftLeft(rootNode);
  }

  rotateRightLeft(rootNode) {
    // 오른쪽 노드가 교체될 예정이므로 rootNode에서 노드를 분리
    const rightNode = rootNode.right;
    rootNode.setRight(null);
    // rightNode에서 왼쪽 노드 분리
    const rightLeftNode = rightNode.left;
    rightNode.setLeft(null);
    if (rightLeftNode.right) {
      rightNode.setLeft(rightLeftNode.right);
      rightLeftNode.setRight(null);
    }
    // rootNode에 rightLeftNode 붙이기
    rootNode.setRight(rightLeftNode);
    // rightNode를 rightLeftNode의 오른쪽 노드로 붙이기
    rightLeftNode.setRight(rightNode);
    // 오른쪽-오른쪽 회전 실행
    this.rotateRightRight(rootNode);
  }

  rotateRightRight(rootNode) {
    // 루트 노드(rootNode)에서 오른쪽 노드(rightNode)를 분리
    const rightNode = rootNode.right;
    rootNode.setRight(null);
    // rightNode를 rootNode 부모의 자식 노드로 만든다
    if (rootNode.parent) {
      rootNode.parent.setRight(rightNode);
    } else if (rootNode === this.root) {
      // 루트 노드가 루트인 경우, rightNode를 새 루트로 만든다
      this.root = rightNode;
    }
    // rightNode에 왼쪽 자식이 있는 경우 이를 분리하여 rootNode의 오른쪽 자식으로 연결
    if (rightNode.left) {
      rootNode.setRight(rightNode.left);
    }
    // Attach rootNode to the left of rightNode.
    // rootNode를 rightNode의 왼쪽에 연결
    rightNode.setLeft(rootNode);
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/AVL_tree)

[Tutorials Point](https://www.tutorialspoint.com/data_structures_algorithms/avl_tree_algorithm.htm)

[BTech](http://btechsmartclass.com/data_structures/avl-trees.html)

[AVL Tree Insertion on YouTube](https://www.youtube.com/watch?v=rbg7Qf8GkQ4&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=12&)

[AVL Tree Interactive Visualisations](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html)
