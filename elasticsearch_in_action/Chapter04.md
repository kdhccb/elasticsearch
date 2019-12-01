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
- 본문 기반 검색 요청을 사용하면, 유연하면서 더 많은 옵션을 제공하는 좀 더 고급 검색을 실행해 볼 수 있다.
- match_all : 모든 도큐먼트 가져오기
- \_source : 반환할 필드 목록을 지정
  - 와일드카드 사용이 가능함. 예를들면, "name"과 "nation" 처럼 "na"로 시작하는 모든 필드를 반환하려면 `\_source : "na*"`를 지정한다.
  - include : 어떤 필드를 포함할지 지정
  - exclude : 어떤 필드를 포함하지 않을 지 지정
![\_source](https://user-images.githubusercontent.com/49108738/69914610-75f09e80-1489-11ea-858f-3457b4a58b32.png)
- sort : 결과에서 순서를 정렬할 때 지정
![범위, 페이지 매김, 필드, 정렬순서 사용](https://user-images.githubusercontent.com/49108738/69914717-8ce3c080-148a-11ea-9706-02bc8343ce7c.png)

#### 4.1.4 응답 구조 이해하기
- title:elasticsearch : title에 `elasticsearch`가 포함된 도큐먼트를 가져온다.
- \_source=title,date : 검색결과로 `title`, `date`필드만 가져온다.
![응답1](https://user-images.githubusercontent.com/49108738/69914730-e1873b80-148a-11ea-95c0-fe8cca45a37d.png)
![응답2](https://user-images.githubusercontent.com/49108738/69914762-46db2c80-148b-11ea-9e24-434b2679bf36.png)

### 4.2 쿼리와 필터 DSL 소개
