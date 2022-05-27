[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/segment-tree/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/segment-tree/README.md)

## **Segment Tree (세그먼트 트리)**

\- a.k.a. 통계 트리

\- 간격 또는 세그먼트에 대한 정보를 저장하는 데 사용되는 트리 데이터 구조

\- 저장된 세그먼트 중 지정된 포인트가 포함된 세그먼트를 쿼리할 수 있다

\- 원칙적으로 정적 구조 (즉, 한 번 빌드되면 수정할 수 없는 구조)

\- 유사한 데이터 구조가 [간격 트리](https://en.wikipedia.org/wiki/Interval_tree)

\- 간격 또는 세그먼트 의 집합 I 이 주어지면 I 에 대한 세그먼트 트리 _T_ 는 다음과 같이 구성된다

- T 는 이진 트리
- 그것의 리프는 I 의 끝점에 의해 유도된 기본 간격에 순서대로 대응한다  
  가장 왼쪽 잎은 가장 왼쪽 간격에 대응하는 식  
  잎 v 에 해당하는 기본 간격 은 Int( v )로 표시된다
- T 의 내부 노드는 기본 간격 의 합집합인 간격에 해당한다  
  노드 N 에 해당하는 간격 Int( N )은 N 을 근간으로 하는 트리의 잎에 해당하는 간격의 합집합  
  이는 Int( N )이 두 자식 간격의 합집합임을 의미한다
- T 의 각 노드 또는 리프 v 는 일부 데이터 구조에서 간격 Int( v ) 및 간격 세트를 저장한다  
  노드 v 의 이 정규 부분집합 에는 \[ x , x' \] 에 Int( v )가 포함되고 Int(parent( v ))가 포함되지 않도록 I 의 구간 \[ x , x' \] 가 포함된다  
  즉, T 의 각 노드는 해당 간격에 걸쳐 있지만 부모의 간격을 통해 확장되지 않는 세그먼트를 저장한다

\- 세그먼트 트리는 이진 트리

\- 트리의 루트는 전체 배열을 나타낸다

\- 루트의 두 자식은 배열의 첫 번째, 두 번째 부분을 나타낸다

\- 마찬가지로 각 노드의 자식은 노드에 해당하는 배열의 두 절반에 해당한다

\- 각 노드 값이 자식 값의 "최소"(또는 다른 함수)가 되도록 트리를 상향식으로 구축 (O(n log n) 시간이 걸림)

\- 수행된 작업의 수는 트리의 높이인 O(log n)이다

\- 레인지 쿼리를 수행하기 위해 각 노드는 커리를 자식마다 하나의 하위 쿼리인 두 부분으로 나눈다

\- 만약 쿼리에 노드의 전체 하위 배열이 포함된 경우 노드에서 미리 계산된 값을 사용할 수 있다

\- 이 최적화를 사용하면 O(log n) 최소 연산만 수행된다는 것을 증명할 수 있다

![Min Segment Tree](https://www.geeksforgeeks.org/wp-content/uploads/RangeMinimumQuery.png)![Sum Segment Tree](https://www.geeksforgeeks.org/wp-content/uploads/segment-tree1.png)

### 1) 적용

\- 세그먼트 트리는 특정 어레이 작업(특히 범위 쿼리와 관련된 작업)을 효율적으로 수행하도록 설계된 데이터 구조

\- 세그먼트 트리의 응용은 계산 기하학 및 지리 정보 시스템 영역에 있다

\- 세그먼트 트리의 현재 구현은 당신이 임의의 이진 함수(두 개의 입력 매개 변수로)를 그것에 전달할 수 있다는 것을 의미하며, 따라서 다양한 함수에 대한 범위 쿼리를 할 수 있다

\- 테스트에서 세그먼트 트리에서 "min", "max" 및 "sum" 범위 쿼리를 수행하는 예를 찾을 수 있다

### 2) 자바스크립트로 구현해보기

\- isPowerOfTwo(n) (n이 2의 거듭제곱인지 아닌지 리턴해주는 함수)을 이용한 구현

```javascript
export default function isPowerOfTwo(number) {
  // 1(2^0) 은 2의 최소 거듭제곱
  if (number < 1) {
    return false;
  }
  // 나머지 없이 2배로 나눌 수 있는지
  let dividedNumber = number;
  while (dividedNumber !== 1) {
    if (dividedNumber % 2 !== 0) {
      // 나머지가 0이 아닌 모든 경우에 대해, 우리는 이 숫자가 2의 거듭제곱의 결과일 수는 없다고 말할 수 있음
      return false;
    }
    dividedNumber /= 2;
  }
  return true;
}

export default class SegmentTree {
  constructor(inputArray, operation, operationFallback) {
    this.inputArray = inputArray;
    this.operation = operation;
    this.operationFallback = operationFallback;
    // 세그먼트 트리 초기 배열 표현
    this.segmentTree = this.initSegmentTree(this.inputArray);
    this.buildSegmentTree();
  }

  initSegmentTree(inputArray) {
    let segmentTreeArrayLength;
    const inputArrayLength = inputArray.length;
    if (isPowerOfTwo(inputArrayLength)) {
      // 원래 배열 길이가 2의 거듭제곱인 경우
      segmentTreeArrayLength = (2 * inputArrayLength) - 1;
    } else {
      // 원래 배열 길이가 2의 거듭제곱이 아니라면 2의 거듭제곱인 다음 수를 찾아 트리 배열 크기를 계산해야 함
      // 완벽한 이진 트리의 빈 하위 항목을 null로 채워야 하기 때문에 (Null은 추가 공간이 필요)
      const currentPower = Math.floor(Math.log2(inputArrayLength));
      const nextPower = currentPower + 1;
      const nextPowerOfTwoNumber = 2 ** nextPower;
      segmentTreeArrayLength = (2 * nextPowerOfTwoNumber) - 1;
    }
    return new Array(segmentTreeArrayLength).fill(null);
  }

  // 세그먼트 트리 빌드
  buildSegmentTree() {
    const leftIndex = 0;
    const rightIndex = this.inputArray.length - 1;
    const position = 0;
    this.buildTreeRecursively(leftIndex, rightIndex, position);
  }

  // 세그먼트 트리를 반복적으로 작성
  buildTreeRecursively(leftInputIndex, rightInputIndex, position) {
    // 낮은 인풋 인덱스와 높은 인풋 인덱스가 같으면 분할이 완료되었고
    // 이미 세그먼트 트리의 리프에 도달했음을 의미한다
    // 이 리프 값을 입력 배열에서 세그먼트 트리로 복사해야 한다
    if (leftInputIndex === rightInputIndex) {
      this.segmentTree[position] = this.inputArray[leftInputIndex];
      return;
    }

    // 입력 배열을 두 개로 나누고 재귀적으로 처리
    const middleIndex = Math.floor((leftInputIndex + rightInputIndex) / 2);
    // 입력 배열의 왼쪽 절반을 처리
    this.buildTreeRecursively(leftInputIndex, middleIndex, this.getLeftChildIndex(position));
    // 입력 배열의 오른쪽 절반 처리
    this.buildTreeRecursively(middleIndex + 1, rightInputIndex, this.getRightChildIndex(position));
    // Once every tree leaf is not empty we're able to build tree bottom up using
    // provided operation function.
    // 모든 트리 리프가 비어 있지 않으면 제공된 작동 함수를 사용하여 트리 바텀업을 구축할 수 있다
    // bottom-up: 각 재귀 level에서, 먼저 모든 자식 node들에 대해 재귀적으로 함수를 호출한 뒤 리턴값과 root node의 값에 따라 정답을 결정하는 방식
    this.segmentTree[position] = this.operation(
      this.segmentTree[this.getLeftChildIndex(position)],
      this.segmentTree[this.getRightChildIndex(position)],
    );
  }

  // this.opeartion 함수의 컨텍스트에서 세그먼트 트리에 대한 레인지 쿼리 실행
  rangeQuery(queryLeftIndex, queryRightIndex) {
    const leftIndex = 0;
    const rightIndex = this.inputArray.length - 1;
    const position = 0;
    return this.rangeQueryRecursive(
      queryLeftIndex,
      queryRightIndex,
      leftIndex,
      rightIndex,
      position,
    );
  }

  // this.operation 함수의 컨텍스트에서 세그먼트 트리에 대한 범위 쿼리를 재귀적으로 실행
  rangeQueryRecursive(queryLeftIndex, queryRightIndex, leftIndex, rightIndex, position) {
    if (queryLeftIndex <= leftIndex && queryRightIndex >= rightIndex) {
      // 전체 오버랩
      return this.segmentTree[position];
    }
    if (queryLeftIndex > rightIndex || queryRightIndex < leftIndex) {
      // 오버랩 없음
      return this.operationFallback;
    }
    // 부분 오버랩
    const middleIndex = Math.floor((leftIndex + rightIndex) / 2);
    const leftOperationResult = this.rangeQueryRecursive(
      queryLeftIndex,
      queryRightIndex,
      leftIndex,
      middleIndex,
      this.getLeftChildIndex(position),
    );
    const rightOperationResult = this.rangeQueryRecursive(
      queryLeftIndex,
      queryRightIndex,
      middleIndex + 1,
      rightIndex,
      this.getRightChildIndex(position),
    );
    return this.operation(leftOperationResult, rightOperationResult);
  }

  // 왼쪽 자식 인덱스
  getLeftChildIndex(parentIndex) {
    return (2 * parentIndex) + 1;
  }

  // 오른쪽 자식 인덱스
  getRightChildIndex(parentIndex) {
    return (2 * parentIndex) + 2;
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Segment_tree)  
[YouTube](https://www.youtube.com/watch?v=ZBHKZF5w4YU&index=65&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)  
[GeeksForGeeks](https://www.geeksforgeeks.org/segment-tree-set-1-sum-of-given-range/)
