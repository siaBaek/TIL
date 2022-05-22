[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/trie/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/tree/README.md)

## **Trie (트라이)**

\- re**trie**val(검색)에서 유래하였고,  tree와 구분하기 위하여 **트라이**라고 발음하는 사람이 많다고 한다

\- 디지털 트리(digital tree), 기수 트리(radix tree), 접두사 트리(prefix tree), prefixes 라고도 한다

\- **문자열을 저장**하고 **효율적으로 탐색**하기 위한 **트리 형태**의 자료구조이다

\- 많은 양의 텍스트정보를 빠르고 효율적으로 검색하기 위해 사용한다

\- 사전 혹은 인터넷 자동완성의 retrieval을 효과적으로 할 수 있는 자료구조이다

\- 기본적으로 k진 트리(k-ary tree) 구조를 띠고 있다

\- 일종의 검색 트리로, 키가 보통 문자열인 동적 셋이나 연관 배열을 저장하기 위해 사용되는 정렬된 트리 데이터 구조이다

\- 이진 검색 트리와 달리, 트리의 어떤 노드도 해당 노드와 연관된 키를 저장하지 않는 대신에 **트리에서의 노드 위치가 연관된 키를 정의**한다

\- 이것은 데이터 구조 전체에 걸쳐 각 키의 값을 분배하며 모든 노드에 반드시 연관된 값이 있는 것이 있는 것은 아니다 (값이 모든 노드와 반드시 연관되지는 않는다)

\- 오히려 값은 **말단 노드(leaves)**와 특정 키에 해당하는 일부 **내부 노드(inner)**와만 연관되는 경향이 있다

\- 키워드 정보는 말단 노드가 가지고 있고, 나머지 노드들은 정보는 없고 링크만 가지게 된다

\- 노드의 모든 자식 노드는 해당 부모 노드와 연결된 문자열의 **공통 접두사**를 가지며, **루트는 빈 문자열**과 연결된다

\- 접두사로 액세스할 수 있는 데이터를 저장하는 이 작업은 기수 트리를 사용하여 메모리 최적화 방식으로 수행할 수 있다

\- prefix tree의 공간 최적화 표시는 compact prefix tree를 참조

![Trie](https://upload.wikimedia.org/wikipedia/commons/b/be/Trie_example.svg)

### 1) 시간 복잡도

\- 문자열 집합의 개수와 상관 없이 **찾고자 하는 문자열의 길이**가 시간 복잡도가 된다.

\- 문자열의 길이가 **m**이라면 시간 복잡도는 **O(m)**

\- 이진 검색 트리: O(logn)

\- 문자열 이진 검색 트리: O(Mlogn)

- 문자열은 정수와 실수와는 달리 그 길이가 변하기 때문에 검색에 시간이 많이 걸린다
- 길이가 고정된 정수나 실수의 경우 이진 트리에서 O(logn) -> 이진 트리에서는 한 번 검색할 때마다 대상이 반으로 줄기 때문
- 문자열은 길이가 변하기 때문에 문자열의 최대 길이 M을 곱한 O(MlogN)이 된다

**m: 문자열의 길이**

**n: 문자열 집합의 개수**

**트라이를 구현할 때, n과 m이 상대적으로 작다면 배열을 사용할텐데, 현재 노드의 위치를 i, j번째 문자라고 한다면 trie\[i\]\[j\] 위치를 조회하는 건 O(1)에 가능하다. 여기서 m번을 수행하니 O(m)의 시간이 걸리는 것이다.**

**하지만 n과 m이 큰 경우에는, 메모리 확보를 위해 시간을 희생하더라도 std::map으로 트라이를 만들기도 한다. 이 경우엔 O(mlogn)의 시간이 소모된다.**

### 2) 자바스크립트로 구현해보기

\- [해시 테이블](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/hash_table.md) 이용하기

```javascript
// TrieNode
import HashTable from '../hash-table/HashTable'; // 이전 포스팅 참고

export default class TrieNode {
  constructor(character, isCompleteWord = false) {
    this.character = character;
    this.isCompleteWord = isCompleteWord;
    this.children = new HashTable();
  }

  getChild(character) {
    return this.children.get(character);
  }

  addChild(character, isCompleteWord = false) {
    if (!this.children.has(character)) {
      this.children.set(character, new TrieNode(character, isCompleteWord));
    }
    const childNode = this.children.get(character);
    // carpet 뒤 car를 추가하는 것과 비슷한 경우, "r" 문자를 완료로 표시해야 한다
    childNode.isCompleteWord = childNode.isCompleteWord || isCompleteWord;
    return childNode;
  }

  removeChild(character) {
    const childNode = this.getChild(character);
    // 다음의 경우에만 childNode 삭제
    // - childNode가 자식이 없는 경우
    // - childNode.isCompleteWord === false
    if (childNode && !childNode.isCompleteWord && !childNode.hasChildren()) {
      this.children.delete(character);
    }
    return this;
  }

  hasChild(character) {
    return this.children.has(character);
  }

  // 현재 TrieNode에 자식이 있는지 확인합니다.
  hasChildren() {
    return this.children.getKeys().length !== 0;
  }

  suggestChildren() {
    return [...this.children.getKeys()];
  }

  toString() {
    let childrenAsString = this.suggestChildren().toString();
    childrenAsString = childrenAsString ? `:${childrenAsString}` : '';
    const isCompleteString = this.isCompleteWord ? '*' : '';
    return `${this.character}${isCompleteString}${childrenAsString}`;
  }
}
```

```javascript
import TrieNode from './TrieNode';

// 트라이 tree root에 사용할 문자
const HEAD_CHARACTER = '*';

export default class Trie {
  constructor() {
    this.head = new TrieNode(HEAD_CHARACTER);
  }

  addWord(word) {
    const characters = Array.from(word);
    let currentNode = this.head;
    for (let charIndex = 0; charIndex < characters.length; charIndex += 1) {
      const isComplete = charIndex === characters.length - 1;
      currentNode = currentNode.addChild(characters[charIndex], isComplete);
    }
    return this;
  }

  deleteWord(word) {
    const depthFirstDelete = (currentNode, charIndex = 0) => {
      if (charIndex >= word.length) {
        // Return if we're trying to delete the character that is out of word's scope.
        return;
      }
      const character = word[charIndex];
      const nextNode = currentNode.getChild(character);
      if (nextNode == null) {
        // Trie에 추가되지 않은 단어를 삭제하려면 리턴
        return;
      }
      // Go deeper.
      depthFirstDelete(nextNode, charIndex + 1);
      // 단어를 삭제할 것이므로 마지막 문자 isCompleteWord 플래그 표시를 해제
      if (charIndex === word.length - 1) {
        nextNode.isCompleteWord = false;
      }
      // 다음의 경우에만 childNode 삭제
      // - childNode가 자식이 없는 경우
      // - childNode.isCompleteWord === false
      currentNode.removeChild(character);
    };
    // 헤드 노드에서 깊이 우선 삭제를 시작
    depthFirstDelete(this.head);
    return this;
  }

  suggestNextCharacters(word) {
    const lastCharacter = this.getLastCharacterNode(word);
    if (!lastCharacter) {
      return null;
    }
    return lastCharacter.suggestChildren();
  }

  // Trie에 완전한 단어가 있는지 확인
  doesWordExist(word) {
    const lastCharacter = this.getLastCharacterNode(word);
    return !!lastCharacter && lastCharacter.isCompleteWord;
  }

  getLastCharacterNode(word) {
    const characters = Array.from(word);
    let currentNode = this.head;
    for (let charIndex = 0; charIndex < characters.length; charIndex += 1) {
      if (!currentNode.hasChild(characters[charIndex])) {
        return null;
      }
      currentNode = currentNode.getChild(characters[charIndex]);
    }
    return currentNode;
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Trie)  
[YouTube](https://www.youtube.com/watch?v=zIjfhVPRZCg&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8&index=7&t=0s)
