[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/priority-queue/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/priority-queue/README.ko-KR.md)

## **Priority Queue (우선순위 큐)**

\- 일반 큐 또는 스택 구조 같은 추상 데이터 유형이지만, 각 요소에 **우선 순위**가 연결된다

\- 우선 순위가 높은 요소가 낮은 요소 앞에 제공된다

\- 두 요소가 동일한 우선 순위를 가질 경우 큐의 순서에 따른다

### 1) 자바스크립트로 구현해보기

\- [최소 힙과 comparator](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/heap.md)를 이용한 구현

\- 두 요소를 비교할 때 요소의 값 대신 우선 순위를 고려한다는 점을 제외하면 최소 힙과 동일하다

```javascript
import MinHeap from '../heap/MinHeap';
import Comparator from '../../utils/comparator/Comparator';

export default class PriorityQueue extends MinHeap {
  constructor() {
    // minheap 컨스트럭터를 먼저 호출
    super();
    // Setup priorities map.
    this.priorities = new Map();
    // 엘리먼트 우선순위를 고려하는 힙 엘리먼트를 위해 커스텀 comparator 사용
    this.compare = new Comparator(this.comparePriority.bind(this));
  }

  // 아이템을 우선 순위 큐에 추가
  add(item, priority = 0) {
    this.priorities.set(item, priority);
    super.add(item);
    return this;
  }

  // 아이템을 우선 순위 큐에서 삭제
  remove(item, customFindingComparator) {
    super.remove(item, customFindingComparator);
    this.priorities.delete(item);
    return this;
  }

  // 큐에서 아이템의 우선순위를 변경
  changePriority(item, priority) {
    this.remove(item, new Comparator(this.compareValue));
    this.add(item, priority);
    return this;
  }

  // value로 아이템 찾기
  findByValue(item) {
    return this.find(item, new Comparator(this.compareValue));
  }

  // 아이템이 큐에 있는지 확인
  hasValue(item) {
    return this.findByValue(item).length > 0;
  }

  // 두 아이템의 우선순위를 비교
  comparePriority(a, b) {
    if (this.priorities.get(a) === this.priorities.get(b)) {
      return 0;
    }
    return this.priorities.get(a) < this.priorities.get(b) ? -1 : 1;
  }

  // 두 아이템의 value를 비교
  compareValue(a, b) {
    if (a === b) {
      return 0;
    }
    return a < b ? -1 : 1;
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Priority_queue)  
[YouTube](https://www.youtube.com/watch?v=wptevk0bshY&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=6)
