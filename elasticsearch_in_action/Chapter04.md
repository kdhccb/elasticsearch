## Chapter04 데이터 검색
- 일래스틱서치의 검색 요청과 응답 구조
- 일래스틱서치 필터 및 필터와 쿼리 차이점
- 필터 비트셋(Bitset)과 캐시(Caching)
- 일래스틱서치가 제공하는 쿼리와 필터 사용하기

### 4.1 검색 요청의 구조
- REST API 검색 요청은 처음 접속하려고 선택한 노드에 전송되고 검색 요청을 모든 샤드(주 또는 레플리카)로 보낸다.
![검색 요청의 구조](https://user-images.githubusercontent.com/49108738/69907368-705a7080-1417-11ea-8085-c481708165bc.png)


#### 4.1.1 검색 범위 지정하기
- curl 'localhost:9200/`+get-toge*`,`-get-together`/\_search'-d '...'
  - get-together 색인이 아닌 get-toge로 시작하는 모든 색인에서 검색
- alias : 다중의 index에 하나의 alias를 부여하여, 하나의 index처럼 사용가능하도록 함
![alias](https://user-images.githubusercontent.com/49108738/69907439-06db6180-1419-11ea-8869-42760969f843.png)

#### 4.1.2 검색 요청의 기본 구성 요소
- 반환할 도큐먼트 개수를 제어 : size
- 최적의 도큐먼트를 선택 : score, sort, from, ...
  - from : list타입의 index와 비슷한 의미, 예를들면 `from:10`은 9번째 문서부터 선택한다는 의미이다.(첫번째 문서의 index는 0이다.)
- 원치 않는 도큐먼트는 결과에서 걸래냄 : filter, bool, ...
![URL기반 검색 요청](https://user-images.githubusercontent.com/49108738/69907556-4d7d8b80-141a-11ea-8dfb-ce0f2cc3df14.png)

#### 4.1.3 본문 기반 검색 요청 사용하기
- DSL : Domain Specific Language
