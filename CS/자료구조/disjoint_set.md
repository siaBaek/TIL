[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/disjoint-set/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/disjoint-set/README.md)

## **Disjoint-set (서로소 집합)**

\- union-find 데이터 구조 또는 merge-find set 이라고도 한다

\-  여러 개의 서로소 (서로 중복되지 않는) 부분 집합으로 분할된 요소 집합을 추적하는 데이터 구조

\- 새로운 집합을 추가하고, 기존 집합을 병합하고, 요소가 동일한 집합에 있는지 여부를 결정하기 위해 역 애커만 함수에 의해 제한되는 거의 일정한 시간(near-constant-time) 연산을 제공한다

\- 많은 다른 용도(응용프로그램 섹션 참조) 외에도, 서로소 집합은 그래프의 최소 스패닝 트리를 찾는 크루스칼 알고리즘에서 중요한 역할을 한다

![disjoint set](https://upload.wikimedia.org/wikipedia/commons/6/67/Dsu_disjoint_sets_init.svg)

MakeSet creates 8 singletons.

![disjoint set](https://upload.wikimedia.org/wikipedia/commons/a/ac/Dsu_disjoint_sets_final.svg)

After some operations of Union, some sets are grouped together.

### 1) 자바스크립트로 구현해보기

```javascript
// DisjointSetItem

class DisjointSetItem {
  constructor(value, keyCallback) {
    this.value = value;
    this.keyCallback = keyCallback;
    this.parent = null;
    this.children = {};
  }

  getKey() {
    // 사용자가 사용자 지정 키 제너레이터를 정의
    if (this.keyCallback) {
      return this.keyCallback(this.value);
    }
    // 그렇지 않으면 기본적으로 값을 키로 사용
    return this.value;
  }

  getRoot() {
    return this.isRoot() ? this : this.parent.getRoot();
  }

  isRoot() {
    return this.parent === null;
  }

  // Rank는 기본적으로 모든 조상의 수를 의미
  getRank() {
    if (this.getChildren().length === 0) {
      return 0;
    }
    let rank = 0;
    this.getChildren().forEach((child) => {
      // 자식 카운트
      rank += 1;
      // 또한 현재 자식의 모든 자식들을 추가
      rank += child.getRank();
    });
    return rank;
  }

  getChildren() {
    return Object.values(this.children);
  }

  setParent(parentItem, forceSettingParentChild = true) {
    this.parent = parentItem;
    if (forceSettingParentChild) {
      parentItem.addChild(this);
    }
    return this;
  }

  addChild(childItem) {
    this.children[childItem.getKey()] = childItem;
    childItem.setParent(this, false);
    return this;
  }
}

// DisjointSet

export default class DisjointSet {
  constructor(keyCallback) {
    this.keyCallback = keyCallback;
    this.items = {};
  }

  makeSet(itemValue) {
    const disjointSetItem = new DisjointSetItem(itemValue, this.keyCallback);
    if (!this.items[disjointSetItem.getKey()]) {
      // 아직 제시되지 않은 경우에만 새 item 추가
      this.items[disjointSetItem.getKey()] = disjointSetItem;
    }
    return this;
  }

  // 집합 표현 노드 찾기
  find(itemValue) {
    const templateDisjointItem = new DisjointSetItem(itemValue, this.keyCallback);
    // 아이템 찾기
    const requiredDisjointItem = this.items[templateDisjointItem.getKey()];
    if (!requiredDisjointItem) {
      return null;
    }
    return requiredDisjointItem.getRoot().getKey();
  }

  // 랭크별 조합
  union(valueA, valueB) {
    const rootKeyA = this.find(valueA);
    const rootKeyB = this.find(valueB);
    if (rootKeyA === null || rootKeyB === null) {
      throw new Error('One or two values are not in sets');
    }
    if (rootKeyA === rootKeyB) {
      // 두 엘리먼트가 이미 동일한 집합이 있는 경우, 키를 반환
      return this;
    }
    const rootA = this.items[rootKeyA];
    const rootB = this.items[rootKeyB];
    if (rootA.getRank() < rootB.getRank()) {
      // rootB 트리가 더 크면 rootB를 새 root로 만든다
      rootB.addChild(rootA);
      return this;
    }
    // rootA 트리가 더 크면 rootA를 새 root로 만든다
    rootA.addChild(rootB);
    return this;
  }

  inSameSet(valueA, valueB) {
    const rootKeyA = this.find(valueA);
    const rootKeyB = this.find(valueB);
    if (rootKeyA === null || rootKeyB === null) {
      throw new Error('One or two values are not in sets');
    }
    return rootKeyA === rootKeyB;
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)  
[By Abdul Bari on YouTube](https://www.youtube.com/watch?v=wU6udHRIkcc&index=14&t=0s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
