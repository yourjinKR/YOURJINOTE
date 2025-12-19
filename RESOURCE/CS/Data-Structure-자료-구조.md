# Data Structure

### [MDN](https://developer.mozilla.org/ko/docs/Glossary/Data_structure)
데이터 구조는 '데이터'를 효율적으로 사용할 수 있도록 구성하는 특별한 방법이다.
### [Wikipedia](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0#cite_note-5)
[컴퓨터 과학](CS.md)에서 효율적인 접근 및 수정을 가능케 하는 자료의 조직, 관리, 저장을 의미한다.  
자료 구조는 데이터 값의 모임, 데이터 간의 관계, 그리고 데이터에 적용할 수 있는 함수나 명령을 의미한다.
### [geeksforgeeks](https://www.geeksforgeeks.org/dsa/data-structure-meaning/)
데이터 구조는 컴퓨터에서 데이터를 효율적으로 접근하고 사용할 수 있도록 데이터를 구성하고 저장하는 방식이다. 이는 데이터의 논리적 또는 수학적 표현뿐 아니라 컴퓨터 프로그램에서의 구현 방식까지 포괄한다.

# 분류

### 형태에 따른

- 선형 구조
	- 스택
	- 큐
		- 환형 큐
- 비선형 구조
	- 그래프
		- 유향 그래프, 무향 그래프
	- 트리
		- 이진 트리
			- 힙

### 구현에 따른

- 배열
- 튜플
- 연결 리스트
- 원형 연결 리스트
- 이중 연결 리스트
- 환형 이중 연결 리스트
- 해시 테이블

# 공부 체크

-  **자료구조와 알고리즘의 중요성 (웹 개발 관점)**
    
-  **시간 복잡도 및 공간 복잡도 (Big-O Notation)**
	- [x] 시간 복잡도
	- [ ] 공간 복잡도
    - [ ]  O(1), O(log n), O(n), O(n log n), O(n^2) 등의 개념 이해
    - [ ]  각 자료구조/알고리즘의 시간/공간 복잡도 분석 연습

-  **기본 자료구조**
    
    - [ ]  **배열 (Array)**
        - [x]  기본 개념 및 특징
        - [ ]  동적 배열 (ArrayList 등)의 동작 방식 및 시간 복잡도
    - [ ]  **연결 리스트 (Linked List)**
        - [x]  단일 연결 리스트 (Singly Linked List)
        - [x]  이중 연결 리스트 (Doubly Linked List)
        - [ ]  삽입, 삭제, 검색 연산 및 시간 복잡도
    - [x]  **배열 vs 연결 리스트 비교**
        - [x]  각 자료구조의 장단점 및 사용 사례
    - [ ]  **스택 (Stack)**
        - [ ]  LIFO (Last-In, First-Out) 개념
        - [ ]  주요 연산 (Push, Pop, Peek) 및 시간 복잡도
        - [ ]  웹 개발에서의 활용 예시 (함수 호출 스택 등)
    - [ ]  **큐 (Queue)**
        - [ ]  FIFO (First-In, First-Out) 개념
        - [ ]  주요 연산 (Enqueue, Dequeue, Peek) 및 시간 복잡도
        - [ ]  웹 개발에서의 활용 예시 (요청 큐, 비동기 처리 등)
    - [ ]  **해시 테이블 (Hash Table) / 해시 맵 (Hash Map)**
        - [ ]  키-값 쌍 저장 방식
        - [ ]  해시 함수, 충돌 (Collision) 개념
        - [ ]  충돌 해결 기법 (Chaining, Open Addressing)
        - [ ]  주요 연산 (삽입, 삭제, 검색) 및 평균/최악 시간 복잡도
        - [ ]  **매우 중요!** 웹 개발에서의 광범위한 활용 (캐싱, 세션 관리, 데이터 검색 등)
    - [ ]  **트리 (Tree)**
        - [ ]  기본 용어 (루트, 노드, 간선, 깊이, 높이 등)
        - [ ]  이진 트리 (Binary Tree)
        - [ ]  이진 탐색 트리 (Binary Search Tree, BST)
            - [ ]  검색, 삽입, 삭제 연산 및 시간 복잡도
        - [ ]  균형 이진 탐색 트리 (Balanced BST - 개념적 이해)
            - [ ]  AVL 트리, 레드-블랙 트리 (내부적으로 사용되는 경우)
        - [ ]  트라이 (Trie)
            - [ ]  문자열 검색, 자동 완성 등에서의 활용
-  **핵심 알고리즘**
    
    - [ ]  **정렬 알고리즘 (Sorting Algorithms)**
        - [ ]  버블 정렬, 선택 정렬, 삽입 정렬 (기본 개념 및 O(n^2))
        - [ ]  퀵 정렬, 병합 정렬 (효율적인 O(n log n) 알고리즘 원리)
    - [ ]  **탐색 알고리즘 (Searching Algorithms)**
        - [ ]  선형 탐색 (Linear Search)
        - [ ]  **이진 탐색 (Binary Search)**
            - [ ]  **매우 중요!** 정렬된 데이터에서의 O(log n) 검색
            - [ ]  구현 및 시간 복잡도
    - [ ]  **그래프 알고리즘 (Graph Algorithms)**
        - [ ]  그래프 기본 개념 (정점, 간선, 인접 행렬, 인접 리스트)
        - [ ]  깊이 우선 탐색 (DFS - Depth-First Search)
            - [ ]  개념 및 구현 (스택 활용)
            - [ ]  웹 개발에서의 활용 예시 (크롤링, 경로 찾기 등)
        - [ ]  너비 우선 탐색 (BFS - Breadth-First Search)
            - [ ]  개념 및 구현 (큐 활용)
            - [ ]  웹 개발에서의 활용 예시 (최단 경로, 연결 요소 찾기 등)
-  **실전 문제 풀이**
    
    - [ ]  프로그래머스, 백준 온라인 저지 등에서 관련 문제 풀이
    - [ ]  해시 테이블, 이진 탐색, DFS/BFS 관련 문제 집중 풀이
    - [ ]  웹 개발 시나리오에 적용 가능한 문제 탐색

# 출처
[위키백과](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0#cite_note-5)  
[geeksforgeeks](https://www.geeksforgeeks.org/dsa/data-structure-meaning/)