[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/fenwick-tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/fenwick-tree/README.md)

## **Fenwick Tree (펜윅 트리)**

\- 이진 인덱스 트리 (binary indexed tree) 라고도 한다

\- 요소를 효율적으로 업데이트하고 숫자 테이블에서 접두사 합계를 계산할 수 있는 데이터 구조

\- 편평한 수의 배열과 비교할 때, 펜윅 트리는 두 연산 사이에서 훨씬 더 나은 균형을 이룬다

\- 평평한 숫자 배열과 비교할 때 펜윅 트리는 요소 업데이트와 접두사 합계 계산이라는 두 가지 작업 간에 훨씬 더 나은 균형을 이룬다

\- n개 숫자의 평평한 배열에서 요소 또는 접두사 합계를 저장할 수 있다

\- 첫 번째 경우 접두사 합계를 계산하려면 선형 시간이 필요하고,  
\- 두 번째 경우 배열 요소를 업데이트하려면 선형 시간이 필요하다 (두 경우 모두 다른 연산은 일정한 시간에 수행될 수 있다)

\- 펜윅 트리는 두 작업을 모두 O(log n) 시간에 수행할 수 있도록 한다

\- 이것은 각 노드의 값이 해당 하위 트리의 숫자의 합인 트리로 숫자를 표현함으로써 달성된다

\- 트리 구조를 통해 O(log n) 노드 액세스만 사용하여 작업을 수행할 수 있다

### 1) 구현 참고 사항

\- 이진 인덱스 트리는 배열로 표시된다  
\- 이진 인덱스 트리의 각 노드에는 지정된 배열의 일부 요소의 합이 저장됩니다.  
\- 이진 인덱스 트리의 크기는 n과 같다 (여기서 n은 입력 배열 크기)  
\- 현재 구현에서는 구현의 용이성을 위해 크기를 n+1로 사용  
\- 모든 인덱스는 1 기반

![Binary Indexed Tree](https://www.geeksforgeeks.org/wp-content/uploads/BITSum.png)

![Fenwick Tree](https://upload.wikimedia.org/wikipedia/commons/d/dc/BITDemo.gif)

🔼 하나씩 삽입하여 배열 \[1, 2, 3, 4, 5\]에 대한 이진 인덱스 트리 생성

### 2) 자바스크립트로 구현해보기

```javascript
export default class FenwickTree {
  // 생성자는 'arraySize' 크기의 빈 펜윅 트리를 생성하는데,
  // 인덱스는 1기반이기 때문에 arraySize는 +1 한다
  constructor(arraySize) {
    this.arraySize = arraySize;
    // 트리 배열을 0으로 채운다
    this.treeArray = Array(this.arraySize + 1).fill(0);
  }
  // position의 기존 값에 값을 추가
  increase(position, value) {
    if (position < 1 || position > this.arraySize) {
      throw new Error('Position is out of allowed range');
    }
    for (let i = position; i <= this.arraySize; i += i & -i) {
      this.treeArray[i] += value;
    }
    return this;
  }

  // 인덱스 1에서 position까지의 합계를 쿼리
  query(position) {
    if (position < 1 || position > this.arraySize) {
      throw new Error('Position is out of allowed range');
    }
    let sum = 0;
    for (let i = position; i > 0; i -= i & -i) {
      sum += this.treeArray[i];
    }
    return sum;
  }

  // 인덱스 leftIndex에서 rightIndex까지의 합계를 쿼리
  queryRange(leftIndex, rightIndex) {
    if (leftIndex > rightIndex) {
      throw new Error('Left index can not be greater than right one');
    }
    if (leftIndex === 1) {
      return this.query(rightIndex);
    }
    return this.query(rightIndex) - this.query(leftIndex - 1);
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Fenwick_tree)  
[GeeksForGeeks](https://www.geeksforgeeks.org/binary-indexed-tree-or-fenwick-tree-2/)  
[YouTube](https://www.youtube.com/watch?v=CWDQJGaN1gY&index=18&t=0s&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
