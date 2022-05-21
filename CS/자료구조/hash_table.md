[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/hash-table/README.ko-KR.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/hash-table/README.ko-KR.md)

## **Hash Table (해시 테이블)**

\- 키(key)를 값(value)에 매핑할 수 있는 구조인 *연관 배열*을 구현하는 자료 구조

\- *해시 함수*를 사용해 원하는 값을 담을 수 있는 버킷 또는 슬롯 배열의 인덱스를 계산한다

\- 이상적으로, 해시 함수는 각 키들을 고유 버킷에 할당하지만,

\- 대부분의 해시 테이블은 *불완전한 해시 함수*를 사용하기 때문에

\- 해시 함수를 통해 **두 개 이상의 키에 대해 동일한 인덱스를 생성하는 해시 충돌**이 발생할 수 있다

\- 이러한 해시 충돌은 어떠한 방법으로든 해결되어야 한다.

![Hash Table](https://upload.wikimedia.org/wikipedia/commons/7/7d/Hash_table_3_1_1_0_1_0_0_SP.svg)

\- 다음은 **분리 연결법**을 통해 해시 충돌을 해결한 예시이다.

![Hash Collision](https://upload.wikimedia.org/wikipedia/commons/d/d0/Hash_table_5_0_1_1_1_1_1_LL.svg)

### 1) 자바스크립트로 구현해보기

\- [Linked List](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/singly_linked_list.md) 이용하기

```javascript
import LinkedList from '../linked-list/LinkedList'; // 이전 포스팅 참고

// 해시 테이블 크기는 충돌 횟수에 직접적인 영향을 미친다
// 해시 테이블 사이즈가 클수록 충돌이 줄어든다
// 충돌이 어떻게 처리되는지 보여주기 위하여 해쉬 테이블 사이즈는 작다
const defaultHashTableSize = 32;

export default class HashTable {
  constructor(hashTableSize = defaultHashTableSize) {
    // 특정 크기의 해시 테이블을 만들고, 빈 연결 리스트로 각 버킷을 채운다
    this.buckets = Array(hashTableSize)
      .fill(null)
      .map(() => new LinkedList());
    // 모든 실제 키들의 추적을 빠르게 하기 위해서이다
    this.keys = {};
  }

  hash(key) {
    // 간단한 이유로, 해시를 계산하기 위해 키의 모든 문자의 문자 코드 합계를 사용
    // 아래와 같이 충돌 횟수 줄이기 위해 다항식 문자열 해시 같은 보다 정교한 접근 방식을 사용할 수도 있다
    // hash = charCodeAt(0) * PRIME^(n-1) + charCodeAt(1) * PRIME^(n-2) + ... + charCodeAt(n-1)
    // 여기서 charCodeAt(i)는 키의 i번째 문자 코드이고, n은 키의 길이, PRIME은 31과 같은 소수이다
    const hash = Array.from(key).reduce(
      (hashAccumulator, keySymbol) => hashAccumulator + keySymbol.charCodeAt(0),
      0
    );
    // 해시 테이블 크기에 맞도록 해시 수를 줄인다
    return hash % this.buckets.length;
  }

  set(key, value) {
    const keyHash = this.hash(key);
    this.keys[key] = keyHash;
    const bucketLinkedList = this.buckets[keyHash];
    const node = bucketLinkedList.find({callback: (nodeValue) => nodeValue.key === key});
    if (!node) {
      // Insert new node.
      bucketLinkedList.append({key, value});
    } else {
      // Update value of existing node.
      node.value.value = value;
    }
  }

  delete(key) {
    const keyHash = this.hash(key);
    delete this.keys[key];
    const bucketLinkedList = this.buckets[keyHash];
    const node = bucketLinkedList.find({callback: (nodeValue) => nodeValue.key === key});
    if (node) {
      return bucketLinkedList.delete(node.value);
    }
    return null;
  }

  get(key) {
    const bucketLinkedList = this.buckets[this.hash(key)];
    const node = bucketLinkedList.find({callback: (nodeValue) => nodeValue.key === key});
    return node ? node.value.value : undefined;
  }

  has(key) {
    return Object.hasOwnProperty.call(this.keys, key);
  }

  getKeys() {
    return Object.keys(this.keys);
  }

  getValues() {
    return this.buckets.reduce((values, bucket) => {
      const bucketValues = bucket.toArray().map((linkedListNode) => linkedListNode.value.value);
      return values.concat(bucketValues);
    }, []);
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Hash_table)  
[YouTube](https://www.youtube.com/watch?v=shs0KM3wKv8&index=4&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
