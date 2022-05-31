[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/bloom-filter/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/bloom-filter/README.md)

## **Bloom Filter (블룸 필터)**

\- 한 요소가 집합에 존재하는지 여부를 테스트하기 위해 설계된 공간 효율적인 확률적 데이터 구조

\- 잠재적인 **거짓 양성**을 비용으로 엄청나게 빠르고 최소한의 메모리를 사용하도록 설계되었다

- 거짓 양성(false positive, 1종오류): 실제로는 음성인데 검사 결과는 양성이라고 나오는 것  
  (예: 어떤 메일이 스팸 메일인지 검사하는 프로그램에서, 어떤 메일이 실제로는 스팸 메일이 아니지만 프로그램 검사 결과 스팸 메일이라고 판정하는 것)
- 거짓 음성(false negative, 2종오류): 실제로는 양성인데 검사 결과는 음성이라고 나오는 것  
  (예: 어떤 메일이 스팸 메일인지 검사하는 프로그램에서, 어떤 메일이 실제로는 스팸 메일임에도 불구하고 프로그램 검사 결과 스팸 메일이 아니라고 판정하는 것)

\- 블룸 필터에 의해 어떤 원소가 집합에 속한다고 판단된 경우 실제로는 원소가 집합에 속하지 않는 거짓 양성이 발생하는 것이 가능하지만, 반대로 원소가 집합에 속하지 않는 것으로 판단되었는데 실제로는 원소가 집합에 속하는 거짓 음성은 절대로 발생하지 않는다는 특성이 있다

\-> 거짓 양성 일치는 가능하지만, 거짓 음성은 불가능하다 (즉 쿼리는 "setable in"또는 "definally not in set"을 반환한다)

\- 집합에 원소를 추가하는 것은 가능하나, 집합에서 원소를 삭제하는 것은 불가능하다 (블룸 필터 변형 카운팅으로 해결 가능)

\- 집합 내 원소의 숫자가 증가할수록 거짓 양성 발생 확률도 증가한다

\- 기존의 오류 없는 해싱 기술을 적용할 경우, 소스 데이터 양이 비실용적으로 많은 양의 메모리를 필요로하는 애플리케이션을 위한 기술을 제안

### 1) 알고리즘 설명

\- 블룸 필터는 m비트 크기의 비트 배열 구조를 가진다

\- 또한 블룸 필터에서는 k가지의 서로 다른 해시 함수를 사용하며, 각 해시 함수는 입력된 원소에 대해 m가지의 값을 균등한 확률로 출력해야 한다

\- 일반적으로 k는 추가할 요소의 수에 비례하는 m보다 훨씬 작은 상수이다 (k의 정확한 선택과 m의 비례 상수는 필터의 의도된 거짓 양성률에 의해 결정된다)

**다음은 집합 {x,y,z}를 나타내는 블룸 필터의 예**  
\- 색상 화살표는 각 설정 요소가 매핑되는 비트 배열의 위치를 나타낸다

\- 요소 w는 0을 포함하는 하나의 비트 배열 위치로 해시되기 때문에 집합 {x,y,z}에 포함되지 않는다.

\- 블룸 필터(m=18,k=3)에 세 원소 x,y,z가 추가되어 있다

\- w의 세 해시값 중에서 블룸 필터 값이 0인 경우가 존재하기 때문에, 해당 값은 집합에 속하지 않는다고 판단할 수 있다

![Bloom Filter](https://upload.wikimedia.org/wikipedia/commons/a/ac/Bloom_filter.svg)

\- 원소를 추가하는 경우, 추가하려는 원소에 대해 k가지의 해시 값을 계산한 다음, 각 해시 값에 대응하는 비트를 1로 설정한다

\- 원소를 검사하는 경우, 해당 원소에 대해 k가지의 해시 값을 계산한 다음, 각 해시 값에 대응하는 비트값을 읽는다.

모든 비트가 1인 경우 속한다고 판단하며, 나머지는 속하지 않는다고 판단한다

### 2) 연산

\- 블룸 필터는 삽입 및 검색의 두 가지 주요 연산을 수행할 수 있다.  
\- 검색 결과 거짓 양성이 발생할 수 있다  
\- 삭제 연산은 할 수 없다  
\- 즉, 필터가 항목을 가져올 수 있다  
\- 이전에 항목이 삽입되었는지 확인하러 가면 "no" 또는 "maybe"를 알 수 있다  
\- 삽입 및 검색은 모두 O(1) 작업

### 3) 필터 만들기

\- 블룸 필터는 특정 크기를 할당하여 생성된다  
\- 이 예에서는 **100**을 기본 길이로 사용  
\- 모든 위치가 **false**로 초기화된다

#### 삽입

\- 삽입하는 동안, 많은 해시 함수(지금은 3개의 해시 함수)가 입력의 해시를 만드는 데 사용된다  
\- 이러한 해시 함수는 인덱스를 출력한다  
\- 수신되는 모든 인덱스에서 블룸 필터의 값을 true로 변경한다

#### 삭제

\- 검색하는 동안, 동일한 해시 함수들이 호출되어 입력을 해시하는데 사용된다

\- 그런 다음 수신된 모든 인덱스가 블룸 필터 내부에 **true** 값을 가지고 있는지 확인

\- 모두 **true** 값을 가지고 있다면, 블룸 필터에 이전에 갑싱 삽입되었을 수 있다

\- 그러나, 이전에 삽입된 다른 값이 **true**로 반전되었을 수 있기 때문에 확실하지 않다

\- 값이 반드시 현재 검색중인 항목 때문에 **true**일 필요는 없다

\- 단 하나의 항목만 이전에 삽입되지 않는 한, 절대적인 확실성이 불가능하다

\- 우리의 해시함수에 의해 반환된 인덱스에 대한 블룸 필터를 확인하는 동안, 그 중 하나라도 **false**의 값을 가지고 있다면, 우리는 확실히 그 항목이 이전에 삽입되지 않았다는 것을 알 수 있다

### 4) 거짓 양성

\- 거짓 양성 확률은 세 가지 요인에 의해 결정된다 : 사용하는 해시 함수의 수, 블룸 필터의 크기, 필터에 삽입된 항목의 수

\- 거짓 양성 확률 계산식 : ( 1 - e -kn/m ) k

`k` = 사용하는 해시 함수의 수

`m` = 블룸 필터의 크기

`n` = 필터에 삽입된 항목의 수

\- 변수 k, m, n은 허용 가능한 거짓 양성 정도에 따라 선택해야한다

\- 값을 선택하고 결과 확률이 너무 높으면 값을 조정하고 확률을 다시 계산해야 한다

### 5) 적용

\- 블룸 필터는 블로그 웹 사이트에서 사용할 수 있다  
\- 독자들에게 처음 보는 기사만 보여주는 게 목표라면 블룸 필터가 제격이다.  
\- 아티클을 기반으로 해시 값을 저장할 수 있다  
\- 사용자가 몇 개의 기사를 읽은 후 필터에 삽입할 수 있다  
\- 다음 번에 사용자가 사이트를 방문할 때 해당 기사는 결과에서 필터링될 수 있다  
\- 일부 기사는 실수로 걸러질 수밖에 없겠지만 비용은 받아들일 만하다  
\- 사용자가 사이트를 방문할 때마다 볼 수 있는 새로운 기사가 있는 한 몇 개의 기사를 보지 않아도 괜찮다

### 6) 자바스크립트로 구현해보기

```javascript
export default class BloomFilter {
  constructor(size = 100) {
    // 블룸 필터의 크기가 직접적으로 거짓 양성 가능성에 영향을 준다
    // 크기가 클수록 거짓 양성 가능성은 낮아진다
    this.size = size;
    this.storage = this.createStore(size);
  }

  insert(item) {
    const hashValues = this.getHashValues(item);
    // 각 hasValue 인덱스를 true로 설정
    hashValues.forEach((val) => this.storage.setValue(val));
  }

  mayContain(item) {
    const hashValues = this.getHashValues(item);
    for (let hashIndex = 0; hashIndex < hashValues.length; hashIndex += 1) {
      if (!this.storage.getValue(hashValues[hashIndex])) {
        // 항목이 절대 삽입되지 않았다는 것을 알고 있다
        return false;
      }
    }
    // 항목이 삽입되었거나 삽입되지 않았을 수 있다
    return true;
  }

  // 필터에 대한 데이터 스토어 생성
  // 데이터 자체를 캡슐화하고 필요한 방법에 대한 액세스만 제공하기 위해 이 방법을 사용하여 스토어를 생성
  createStore(size) {
    const storage = [];
    // 모든 인덱스를 false로 초기화
    for (let storageCellIndex = 0; storageCellIndex < size; storageCellIndex += 1) {
      storage.push(false);
    }
    const storageInterface = {
      getValue(index) {
        return storage[index];
      },
      setValue(index) {
        storage[index] = true;
      },
    };
    return storageInterface;
  }

  hash1(item) {
    let hash = 0;
    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) + hash + char;
      hash &= hash; // 32 비트 정수로 변환
      hash = Math.abs(hash);
    }
    return hash % this.size;
  }

  hash2(item) {
    let hash = 5381;
    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) + hash + char; /* hash * 33 + c */
    }
    return Math.abs(hash % this.size);
  }

  hash3(item) {
    let hash = 0;
    for (let charIndex = 0; charIndex < item.length; charIndex += 1) {
      const char = item.charCodeAt(charIndex);
      hash = (hash << 5) - hash;
      hash += char;
      hash &= hash; // 32 비트 정수로 변환
    }
    return Math.abs(hash % this.size);
  }

  // input에 세 개의 모든 해시함수를 실행하고 결과 배열 리턴
  getHashValues(item) {
    return [this.hash1(item), this.hash2(item), this.hash3(item)];
  }
}
```

\[참조\]

[Wikipedia](https://en.wikipedia.org/wiki/Bloom_filter)  
[Bloom Filters by Example](http://llimllib.github.io/bloomfilter-tutorial/)  
[Calculating False Positive Probability](https://hur.st/bloomfilter/?n=4&p=&m=18&k=3)  
[Bloom Filters on Medium](https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff)  
[Bloom Filters on YouTube](https://www.youtube.com/watch?v=bEmBh1HtYrw)
