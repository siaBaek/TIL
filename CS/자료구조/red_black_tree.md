[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/red-black-tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/red-black-tree/README.md)

## **Red-Black Tree (레드-블랙 트리)**

\- 일종의 자가 균형 이진 탐색 트리로, 대표적으로는 연관 배열 등을 구현하는데 쓰이는 자료구조

\- 이진 트리의 각 노드에는 추가 비트가 있으며, 이 비트는 노드의 색상(빨간색 또는 검정색)으로 해석되기도 한다

\- 이러한 색 비트는 삽입 및 삭제 중에 트리가 대략적으로 균형을 유지하도록 하는 데 사용된다

\- 트리의 각 노드를 특정 속성을 만족시키는 방식으로 두 가지 색상 중 하나로 칠함으로써 균형이 유지되며, 이는 최악의 경우 트리가 얼마나 불균형해질 수 있는지를 집합적으로 제한한다

\- 자료의 삽입과 삭제, 검색에서 최악의 경우에도 일정한 실행 시간을 보장한다(worst-case guarantees)

\- [AVL 트리](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/AVL_tree.md)는 레드-블랙 트리보다 더 엄격하게 균형이 잡혀 있기 때문에, 삽입과 삭제를 할 때 최악의 경우에는 더 많은 회전(rotations)이 필요

\- 트리가 수정되면 새 트리가 다시 정렬되고 다시 칠해져 색칠 속성을 복원한다

\- 속성은 이러한 재배열 및 회수가 효율적으로 수행될 수 있도록 설계된다

\- 트리의 균형은 완벽하지 않지만 O(log n) 시간 내에 검색을 보장하기에 충분하다 (n은 트리의 총 요소 수)

\- 삽입 및 삭제 작업 또한 트리 재배열 및 재채색과 함께 O(log n) 시간안에 수행된다

![red-black tree](https://upload.wikimedia.org/wikipedia/commons/6/66/Red-black_tree_example.svg)

🔼 레드-블랙 트리 예시

### 1) 특성

\- 레드-블랙 트리는 각각의 노드가 레드 나 블랙 인 색상 속성을 가지고 있는 이진 탐색 트리  
\- 이진 탐색 트리가 가지고 있는 일반적인 조건에 다음과 같은 추가적인 조건을 만족해야 유효한(valid) 레드-블랙 트리가 된다

1.  노드는 레드 혹은 블랙 중의 하나이다
2.  루트 노드는 블랙이다
3.  모든 리프 노드들(NIL)은 블랙이다
4.  레드 노드의 자식노드 양쪽은 언제나 모두 블랙이다 (즉, 레드 노드는 연달아 나타날 수 없으며, 블랙 노드만이 레드 노드의 부모 노드가 될 수 있다)
5.  어떤 노드로부터 시작되어 그에 속한 하위 리프 노드에 도달하는 모든 경로에는 리프 노드를 제외하면 모두 같은 개수의 블랙 노드가 있다.

\- Some definitions

- 노드의 black depth: 루트부터 노드까지 검정 노드의 수
- black-height: 루트 노드부터 리프 노드에 이르는 모든 경로에서 검은 노드의 균일한 수

\- 이러한 제약조건은 레드-블랙 트리의 중요 특성을 강화한다

- 루트에서 가장 먼 리프까지의 경로는 루트에서 가장 가까운 리프까지의 경로보다 두 배 이상 길지 않다
- 그 결과 트리는 대략적으로 높이가 균형을 이루고 있다
- \- 값을 삽입, 삭제 및 찾는 작업에는 트리의 높이에 비례하는 최악의 시간이 필요하므로, 일반적인 이진 검색 트리와는 달리, 이 이론적인 높이 상한을 통해 레드-블랙 트리는 최악의 경우에 효율적일 수 있다

### 2) 삽입 동안의 균형

### If uncle is RED

![Red Black Tree Balancing](https://www.geeksforgeeks.org/wp-content/uploads/redBlackCase2.png)

### If uncle is BLACK

- Left Left Case (`p` 는 `g` 의 왼쪽 자식이고, `x` 는 `p` 의 왼쪽 자식)
- Left Right Case (`p` 는 `g` 의 왼쪽 자식이고, `x` 는 `p` 의 오른쪽 자식)
- Right Right Case (`p` 는 `g` 의 오른쪽 자식이고, `x` 는`p` 의 오른쪽 자식)
- Right Left Case (`p` 는 `g` 의 오른쪽 자식이고, `x` 는 `p` 의 왼쪽 자식)

#### a. Left Left Case (See g, p and x)

![Red Black Tree Balancing](https://www.geeksforgeeks.org/wp-content/uploads/redBlackCase3a1.png)

#### b. Left Right Case (See g, p and x)

![Red Black Tree Balancing](https://www.geeksforgeeks.org/wp-content/uploads/redBlackCase3b.png)

#### c. Right Right Case (See g, p and x)

![Red Black Tree Balancing](https://www.geeksforgeeks.org/wp-content/uploads/redBlackCase3c.png)

#### d. Right Left Case (See g, p and x)

![Red Black Tree Balancing](https://www.geeksforgeeks.org/wp-content/uploads/redBlackCase3d.png)

### 3) 자바스크립트로 구현해보기

\- [이진탐색트리](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/binary_search_tree.md) 이용하기

```javascript
import BinarySearchTree from '../binary-search-tree/BinarySearchTree'; // 이전 포스팅 참고

// 레드-블랙 트리 노드의 가능한 색상
const RED_BLACK_TREE_COLORS = {
  red: 'red',
  black: 'black',
};

// 노드의 메타 정보의 color property name
const COLOR_PROP_NAME = 'color';

export default class RedBlackTree extends BinarySearchTree {
  insert(value) {
    const insertedNode = super.insert(value);
    if (this.nodeComparator.equal(insertedNode, this.root)) {
      // 항상 루트는 검은색
      this.makeNodeBlack(insertedNode);
    } else {
      // 새로 삽입된 모든 노드 빨간색
      this.makeNodeRed(insertedNode);
    }
    // 모든 조건을 확인하고 노드의 균형을 조정
    this.balance(insertedNode);
    return insertedNode;
  }

  remove(value) {
    throw new Error(`Can't remove ${value}. Remove method is not implemented yet`);
  }

  balance(node) {
    // 루트 노드인 경우 여기서 균형을 맞출 필요가 없다
    if (this.nodeComparator.equal(node, this.root)) {
      return;
    }
    // 부모가 검은색이면 done
    if (this.isNodeBlack(node.parent)) {
      return;
    }
    const grandParent = node.parent.parent;
    if (node.uncle && this.isNodeRed(node.uncle)) {
      // 노드에 빨간색 삼촌이 있으면 리컬러링을 해야 합니다.
      // 부모와 삼촌을 검은색으로 리컬러링
      this.makeNodeBlack(node.uncle);
      this.makeNodeBlack(node.parent);
      if (!this.nodeComparator.equal(grandParent, this.root)) {
        // 루트가 아니라면 부모의 부모를 빨간색으로 리컬러링
        this.makeNodeRed(grandParent);
      } else {
        // 부모의 부모가 검은색 루트라면 아무 것도 하지 않는다
        // 루트는 이미 방금 리컬러링한 두 검은색 형제가 있기 때문
        return;
      }
      // 리컬러링된 부모의 부모가 있는지 더 확인
      this.balance(grandParent);
    } else if (!node.uncle || this.isNodeBlack(node.uncle)) {
      // 만약 노드 삼촌이 검은색이거나 없으면 회전해야한다
      if (grandParent) {
        // 회전 후 우리가 받을 newGrandParent 정의
        let newGrandParent;
        if (this.nodeComparator.equal(grandParent.left, node.parent)) {
          // Left case.
          if (this.nodeComparator.equal(node.parent.left, node)) {
            // Left-left case.
            newGrandParent = this.leftLeftRotation(grandParent);
          } else {
            // Left-right case.
            newGrandParent = this.leftRightRotation(grandParent);
          }
        } else {
          // Right case.
          if (this.nodeComparator.equal(node.parent.right, node)) {
            // Right-right case.
            newGrandParent = this.rightRightRotation(grandParent);
          } else {
            // Right-left case.
            newGrandParent = this.rightLeftRotation(grandParent);
          }
        }
        // 부모가 없는 경우 newGrandParent를 루트로 설정
        if (newGrandParent && newGrandParent.parent === null) {
          this.root = newGrandParent;
          // 루트를 검은색으로 리컬러링
          this.makeNodeBlack(this.root);
        }
        // newGrandParent가 레드-블랙 트리 규칙에 위배되지 않는지 확인
        this.balance(newGrandParent);
      }
    }
  }

  // Left Left Case
  leftLeftRotation(grandParentNode) {
    // 조부모 노드의 부모를 기억
    const grandGrandParent = grandParentNode.parent;
    // grandParentNode가 어떤 형제 타입인지 확인 (왼쪽 또는 오른쪽)
    let grandParentNodeIsLeft;
    if (grandGrandParent) {
      grandParentNodeIsLeft = this.nodeComparator.equal(grandGrandParent.left, grandParentNode);
    }
    // grandParentNode의 왼쪽 노드 기억
    const parentNode = grandParentNode.left;
    // 조부모의 왼쪽 하위 트리로 전송할 것이므로 부모의 오른쪽 노드를 기억
    const parentRightNode = parentNode.right;
    // grandParentNode를 parentNode의 오른쪽 자식 노드로 만든다
    parentNode.setRight(grandParentNode);
    // 하위의 오른쪽 하위 트리를 grandParentNode의 왼쪽 하위 트리로 이동
    grandParentNode.setLeft(parentRightNode);
    // grandParentNode 대신 parentNode 노드를 배치
    if (grandGrandParent) {
      if (grandParentNodeIsLeft) {
        grandGrandParent.setLeft(parentNode);
      } else {
        grandGrandParent.setRight(parentNode);
      }
    } else {
      // 부모 노드를 루트로 설정
      parentNode.parent = null;
    }
    // grandParentNode와 parentNode의 색상을 바꿈
    this.swapNodeColors(parentNode, grandParentNode);
    // 새로운 루트 노드 리턴
    return parentNode;
  }

  // Left Right Case
  leftRightRotation(grandParentNode) {
    // 왼쪽, 왼쪽-오른쪽 노드 기억
    const parentNode = grandParentNode.left;
    const childNode = parentNode.right;
    // 왼쪽 자식 하위 트리를 잃지 않기 위해 왼쪽 노드를 기억
    // 나중에 부모 오른쪽 하위 트리에 다시 할당
    const childLeftNode = childNode.left;
    // parentNode를 childNode 노드의 왼쪽 자식 노드로 만든다
    childNode.setLeft(parentNode);
    // 자식 왼쪽 하위 트리를 부모의 오른쪽 하위 트리로 이동
    parentNode.setRight(childLeftNode);
    // 왼쪽 노드 대신 왼쪽-오른쪽 노드를 배치합니다.
    grandParentNode.setLeft(childNode);
    // 왼쪽-왼쪽 회전 준비 완료
    return this.leftLeftRotation(grandParentNode);
  }

  // Right Right Case
  rightRightRotation(grandParentNode) {
    // 부모와 조부모 노드 기억
    const grandGrandParent = grandParentNode.parent;
    // grandParentNode가 어떤 형제 타입인지 확인 (왼쪽 또는 오른쪽)
    let grandParentNodeIsLeft;
    if (grandGrandParent) {
      grandParentNodeIsLeft = this.nodeComparator.equal(grandGrandParent.left, grandParentNode);
    }
    // grandParentNode의 오른쪽 노드 기억
    const parentNode = grandParentNode.right;
    // 조부모의 오른쪽 하위 트리로 전송할 것이므로 부모의 왼쪽 노드를 기억
    const parentLeftNode = parentNode.left;
    // grandParentNode를 parentNode의 왼쪽 자식 노드로 만든다
    parentNode.setLeft(grandParentNode);
    // 모든 왼쪽 노드를 부모에서 조부모의 오른쪽 하위 트리로 전송
    grandParentNode.setRight(parentLeftNode);
    // grandParentNode 대신 parentNode 노드를 배치
    if (grandGrandParent) {
      if (grandParentNodeIsLeft) {
        grandGrandParent.setLeft(parentNode);
      } else {
        grandGrandParent.setRight(parentNode);
      }
    } else {
      // 부모 노드를 루트로 만들기
      parentNode.parent = null;
    }
    // granparent와 parent 노드의 색상을 바꿈
    this.swapNodeColors(parentNode, grandParentNode);
    // 새로운 루트 노드 리턴
    return parentNode;
  }

  // Right Left Case
  rightLeftRotation(grandParentNode) {
    // 오른쪽, 오른쪽-왼쪽 노드 기억
    const parentNode = grandParentNode.right;
    const childNode = parentNode.left;
    // 오른쪽 하위 트리를 잃지 않기 위해 오른쪽 노드를 기억
    // 나중에 상위 하위 트리에 다시 할당
    const childRightNode = childNode.right;
    // parentNode를 childNode의 오른쪽 자식 노드로 만든다
    childNode.setRight(parentNode);
    // 자식 오른쪽 하위 트리를 부모의 왼쪽 하위 트리로 이동
    parentNode.setLeft(childRightNode);
    // parentNode 대신 childNode 노드를 배치
    grandParentNode.setRight(childNode);
    // 오른쪽-오른쪽 회전 준비 완료
    return this.rightRightRotation(grandParentNode);
  }

  makeNodeRed(node) {
    node.meta.set(COLOR_PROP_NAME, RED_BLACK_TREE_COLORS.red);
    return node;
  }

  makeNodeBlack(node) {
    node.meta.set(COLOR_PROP_NAME, RED_BLACK_TREE_COLORS.black);
    return node;
  }

  isNodeRed(node) {
    return node.meta.get(COLOR_PROP_NAME) === RED_BLACK_TREE_COLORS.red;
  }

  isNodeBlack(node) {
    return node.meta.get(COLOR_PROP_NAME) === RED_BLACK_TREE_COLORS.black;
  }

  isNodeColored(node) {
    return this.isNodeRed(node) || this.isNodeBlack(node);
  }

  swapNodeColors(firstNode, secondNode) {
    const firstColor = firstNode.meta.get(COLOR_PROP_NAME);
    const secondColor = secondNode.meta.get(COLOR_PROP_NAME);
    firstNode.meta.set(COLOR_PROP_NAME, secondColor);
    secondNode.meta.set(COLOR_PROP_NAME, firstColor);
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)  
[Red Black Tree Insertion by Tushar Roy (YouTube)](https://www.youtube.com/watch?v=UaLIHuR1t8Q&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=63)  
[Red Black Tree Deletion by Tushar Roy (YouTube)](https://www.youtube.com/watch?v=CTvfzU_uNKE&t=0s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=64)  
[Red Black Tree Insertion on GeeksForGeeks](https://www.geeksforgeeks.org/red-black-tree-set-2-insert/)  
[Red Black Tree Interactive Visualisations](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)
