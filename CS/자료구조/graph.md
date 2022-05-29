[https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/graph/README.md](https://github.com/trekhleb/javascript-algorithms/blob/master/src/data-structures/graph/README.md)

## **Graph (그래프)**

\- 수학, 특히 그래프 이론의 분야에서 무방향 그래프와 방향 그래프 개념을 구현하기 위한 추상 데이터 유형

\- 그래프 데이터 구조는 무방향 그래프의 경우 이러한 꼭짓점의 정렬되지 않은 쌍 집합 또는 방향 그래프의 경우 순서 쌍 집합과 함께,  
   유한한(및 변경 가능한) 꼭짓점 집합( 노드 또는 점 이라고도 함 )으로 구성된다

\- 이러한 쌍을 무방향 그래프의 경우 edges, arcs 또는 lines이라고 하며, 방향 그래프의 경우 arrows, directed edges, directed arcs 또는 directed lines라고 한다

\- 꼭짓점은 그래프 구조의 일부이거나 정수 인덱스 또는 참조로 표현되는 외부 엔티티일 수 있다

![Graph](https://www.tutorialspoint.com/data_structures_algorithms/images/graph.jpg)

### 1) 자바스크립트로 구현해보기

#### Graph

```javascript
export default class Graph {
  constructor(isDirected = false) {
    this.vertices = {};
    this.edges = {};
    this.isDirected = isDirected;
  }

  addVertex(newVertex) {
    this.vertices[newVertex.getKey()] = newVertex;
    return this;
  }

  getVertexByKey(vertexKey) {
    return this.vertices[vertexKey];
  }

  getNeighbors(vertex) {
    return vertex.getNeighbors();
  }

  getAllVertices() {
    return Object.values(this.vertices);
  }

  getAllEdges() {
    return Object.values(this.edges);
  }

  addEdge(edge) {
    // startVertex, endVertex 찾기
    let startVertex = this.getVertexByKey(edge.startVertex.getKey());
    let endVertex = this.getVertexByKey(edge.endVertex.getKey());
    // 삽입되지 않은 경우 시작 꼭짓점을 삽입
    if (!startVertex) {
      this.addVertex(edge.startVertex);
      startVertex = this.getVertexByKey(edge.startVertex.getKey());
    }
    // 삽입되지 않은 경우 끝 꼭짓점을 삽입
    if (!endVertex) {
      this.addVertex(edge.endVertex);
      endVertex = this.getVertexByKey(edge.endVertex.getKey());
    }
    // 간선(edges)이 이미 추가되었는지 확인
    if (this.edges[edge.getKey()]) {
      throw new Error('Edge has already been added before');
    } else {
      this.edges[edge.getKey()] = edge;
    }
    // 꼭짓점에 간선 추가
    if (this.isDirected) {
      // 방향 그래프 경우 오직 시작 꼭짓점에만 간선 추가
      startVertex.addEdge(edge);
    } else {
      // 무방향 그래프 경우 시작, 끝 꼭짓점에 모두 간선 추가
      startVertex.addEdge(edge);
      endVertex.addEdge(edge);
    }
    return this;
  }

  deleteEdge(edge) {
    // 간선 리스트로부터 간선 삭제
    if (this.edges[edge.getKey()]) {
      delete this.edges[edge.getKey()];
    } else {
      throw new Error('Edge not found in graph');
    }
    // 시작, 끝 꼭짓점 찾아서 간선 삭제
    const startVertex = this.getVertexByKey(edge.startVertex.getKey());
    const endVertex = this.getVertexByKey(edge.endVertex.getKey());
    startVertex.deleteEdge(edge);
    endVertex.deleteEdge(edge);
  }

  findEdge(startVertex, endVertex) {
    const vertex = this.getVertexByKey(startVertex.getKey());
    if (!vertex) {
      return null;
    }
    return vertex.findEdge(endVertex);
  }

  getWeight() {
    return this.getAllEdges().reduce((weight, graphEdge) => {
      return weight + graphEdge.weight;
    }, 0);
  }

  // 방향 그래프의 모든 간선 리버스
  reverse() {
    this.getAllEdges().forEach((edge) => {
      // 그래프와 꼭짓점에서 직선 간선 삭제
      this.deleteEdge(edge);
      // 간선 리버스
      edge.reverse();
      // 그래프와 그래프 꼭짓점에 리버스된 간선 다시 추가
      this.addEdge(edge);
    });
    return this;
  }

  getVerticesIndices() {
    const verticesIndices = {};
    this.getAllVertices().forEach((vertex, index) => {
      verticesIndices[vertex.getKey()] = index;
    });
    return verticesIndices;
  }

  getAdjacencyMatrix() {
    const vertices = this.getAllVertices();
    const verticesIndices = this.getVerticesIndices();
    // 무한대를 가진 init matrix는 한 꼭짓점에서 다른 꼭짓점으로 이동할 방법이 없다는 것을 의미
    const adjacencyMatrix = Array(vertices.length)
      .fill(null)
      .map(() => {
        return Array(vertices.length).fill(Infinity);
      });
    // 컬럼 채우기
    vertices.forEach((vertex, vertexIndex) => {
      vertex.getNeighbors().forEach((neighbor) => {
        const neighborIndex = verticesIndices[neighbor.getKey()];
        adjacencyMatrix[vertexIndex][neighborIndex] = this.findEdge(vertex, neighbor).weight;
      });
    });
    return adjacencyMatrix;
  }

  toString() {
    return Object.keys(this.vertices).toString();
  }
}
```

#### GraphEdge

```javascript
export default class GraphEdge {
  constructor(startVertex, endVertex, weight = 0) {
    this.startVertex = startVertex;
    this.endVertex = endVertex;
    this.weight = weight;
  }

  getKey() {
    const startVertexKey = this.startVertex.getKey();
    const endVertexKey = this.endVertex.getKey();
    return `${startVertexKey}_${endVertexKey}`;
  }

  reverse() {
    const tmp = this.startVertex;
    this.startVertex = this.endVertex;
    this.endVertex = tmp;
    return this;
  }

  toString() {
    return this.getKey();
  }
}
```

#### GraphVertex

\- [연결 리스트](https://github.com/siaBaek/TIL/blob/main/CS/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/singly_linked_list.md) 이용하기

```javascript
import LinkedList from '../linked-list/LinkedList';

export default class GraphVertex {
  constructor(value) {
    if (value === undefined) {
      throw new Error('Graph vertex must have a value');
    }

    const edgeComparator = (edgeA, edgeB) => {
      if (edgeA.getKey() === edgeB.getKey()) {
        return 0;
      }
      return edgeA.getKey() < edgeB.getKey() ? -1 : 1;
    };
    // 일반적으로 문자열 값을 꼭짓점 이름 같이 저장
    // 하지만 보통 어떤 오브젝트나 그럴 수 있다
    this.value = value;
    this.edges = new LinkedList(edgeComparator);
  }

  addEdge(edge) {
    this.edges.append(edge);
    return this;
  }

  deleteEdge(edge) {
    this.edges.delete(edge);
  }

  getNeighbors() {
    const edges = this.edges.toArray();
    const neighborsConverter = (node) => {
      return node.value.startVertex === this ? node.value.endVertex : node.value.startVertex;
    };
    // 시작 또는 끝 꼭짓점 리턴
    // 무방향 그래프의 경우, 현재 꼭짓점이 끝 꼭짓점이 될 수 있다
    return edges.map(neighborsConverter);
  }

  getEdges() {
    return this.edges.toArray().map((linkedListNode) => linkedListNode.value);
  }

  getDegree() {
    return this.edges.toArray().length;
  }

  hasEdge(requiredEdge) {
    const edgeNode = this.edges.find({
      callback: (edge) => edge === requiredEdge,
    });
    return !!edgeNode;
  }

  hasNeighbor(vertex) {
    const vertexNode = this.edges.find({
      callback: (edge) => edge.startVertex === vertex || edge.endVertex === vertex,
    });
    return !!vertexNode;
  }

  findEdge(vertex) {
    const edgeFinder = (edge) => {
      return edge.startVertex === vertex || edge.endVertex === vertex;
    };
    const edge = this.edges.find({callback: edgeFinder});
    return edge ? edge.value : null;
  }

  getKey() {
    return this.value;
  }

  deleteAllEdges() {
    this.getEdges().forEach((edge) => this.deleteEdge(edge));
    return this;
  }

  toString(callback) {
    return callback ? callback(this.value) : `${this.value}`;
  }
}
```

\[참조\]

[Wikipedia](<https://en.wikipedia.org/wiki/Graph_(abstract_data_type)>)  
[Introduction to Graphs on YouTube](https://www.youtube.com/watch?v=gXgEDyodOJU&index=9&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)  
[Graphs representation on YouTube](https://www.youtube.com/watch?v=k1wraWzqtvQ&index=10&list=PLLXdhg_r2hKA7DPDsunoDZ-Z769jWn4R8)
