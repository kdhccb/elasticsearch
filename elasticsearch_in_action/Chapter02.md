# Chapter02 기능 들여다보기
- 문서, 타입, 색인 정의하기
- 일래스틱서치 노드와 주(primary) 및 복제(replica) 샤드 이해하기
- cURL과 데이터 집합으로 문서 색인하기
- 데이터를 찾고 가져오기
- 일래스틱서치 환경설정 옵션 세팅하기
- 다수의 노드로 작업하기

## 2.0 Deprecated contents
- 현재 hombrew 패키지를 통하여 설치하면 elasticsearch version이 `6.8.4`이다. 책에서 소개한 소스코드는 `1.5` 기준이라서 syntax 및 예약어들이 변경이나 deprecated되어 애러가 발생하는 경우가 많다.~~(안 되는게 너무 많다...)~~
- 소스코드 다운로드 6.x
```bash
git clone https://github.com/dakrone/elasticsearch-in-action.git -b 6.x
```
- group 타입이 아니고, `_doc` 타입으로 인덱싱한다.
- cURL put, post 시에 header를 추가한다.(-H, --header <header/@file> Pass custom header(s) to server)
`-H'Content-Type: application/json'`
- `kibana`를 설치하고 검색하자.

## 2.1 논리적인 배치 이해하기: 문서, 타입, 색인
- 애플리케이션과 관리자의 관점에서 본 일래스틱서치 클러스터
![그림2.1](https://user-images.githubusercontent.com/49108738/69000721-77618900-0917-11ea-8141-34b1c6e5c797.png)
- 일래스틱서치에서 데이터의 논리적 배치: 어떻게 애플리케이션이 데이터를 보는가
![그림2.2](https://user-images.githubusercontent.com/49108738/69000782-6ebd8280-0918-11ea-8edf-c3b42310e8c7.png)
- 문서
  - 독립적이다. 문서는 필드(name)와 값(Elasticsearch Denver)을 가지고 있다.
  - 계층을 가질 수 있다. 문서 안의 문서로 생각하자. 필드의 값은 위치 필드의 값이 문자열인 것과 같이 단순형일 수 있다. 다른 필드와 값들을 포함할 수도 있다. 예를들어, 위치 필드는 도시와 거리 주소 모두 포함할 수 있다.
  - 유연한 구조로 되어 있다. 여러분의 문서는 미리 정의한 스키마에 의존하지 않는다. 예를 들어, 모든 이벤트가 서술 값이 필요하지는 않아서 필드가 완전히 생략될 수 있다. 그러나 위치의 위도와 경도 같은 새로운 필드가 필요할지 모른다.
  - 문서는 보통 데이터의 JSON(JavaScript Object Notation) 표현이다.
- 타입
  - 문서에 대한 논리적인 컨테이너다.
  - 각 타입에서 필드의 정의는 매핑이라고 부른다.
  - 매핑은 타입에서 지금까지 색인한 모든 문서의 모든 필드를 포함한다.
  - 새로운 필드가 색인되면 자동으로 타입을 추측하여 매핑에 추가한다.
- 색인
  - 색인은 매핑 타입의 컨테이너다.
  - 각 색인은 `refresh_interval`이라는 설정으로 새로 색인한 문서를 검색할 수 있도록 간격을 정의한다.(default : 초당 1번)
  
## 2.2 물리적 배치 이해하기: 노드와 샤드
- 노드, 주 샤드, 복제(replica) 샤드 이해하기
![그림 2.3](https://user-images.githubusercontent.com/49108738/69001070-f8228400-091b-11ea-9b99-ea024bad21cf.png)
- Default로 일래스틱서치의 각 색인은 5개의 샤드와 1개의 레플리카를 갖는다. 즉, 클러스터가 최소 2개의 노드로 구성되어 있다면, 각 색인은 10개의 샤드가 존재한다.(단, 단일노드의 경우, 레플리카 샤드가 활성화되어 있지 않아서 주 샤드 5개만 활성상태로 존재한다.)
- 스플릿 브레인(split brain)이란? 클러스터를 구성하고 있는 노드 사이에 네트워크에 단절이 발생하여 마스터 후보 노드(master-eligible node)가 각각 마스터노트로 승격하여 하나의 클러스터가 2개의 sub 클러스터로 나뉘어지는 현상이다. 데이터 비동기화 문제가 발생하게 된다.
- 문서의 색인은 다음과 같은 과정으로 만들어진다.
  - 문서 ID의 hash 값에 기반하여 주 샤드에 보내진다.
  - 문서 ID의 hash 값은 주 샤드의 모든 복제 샤드에 색인하도록 보내진다.
- 색인을 검색할 때는 주 샤드와 복제 샤드에 관계없이 검색로드분배를 통해 실행된다.
- TF-IDF 스코어링(일래스틱서치의 기본값)
  - TF(term frequency) :  특정한 단어가 문서 내에 얼마나 자주 등장하는지를 나타내는 값
  - DF(document frequency) : 단어 자체가 문서군 내에서 자주 사용되는 경우, 이것은 그 단어가 흔하게 등장한다는 것을 의미한다. 다시말하면, 단어가 전체 문서 중에서 몇 개의 문서에서 등장했는지가 중요하고, 몇 번 등장했는지는 중요하지 않다. 예를들면 at, in와 같은 전치사는 거의 모든 문서에 등장한다. 이 때 해당 단어는 DF값이 크다.
  - IDF(inverse document frequency) : DF값의 역수, 특정 문서에만 자주 나타나는 단어로 유의미한 단어로 분류할 수 있다.
  - TF-IDF는 문서에 단어가 얼마나 자주 등장하는지, 그리고 문서 전체를 고려했을 때 단어자체가 갖는 특징과 유의미함의 정도가 높은지에 대한 곱연산으로 스코어링 한다.

## 2.3 새로운 데이터 색인
- cURL : 다양한 통신 프로토콜을 이용하여 데이터를 전송하기 위한 라이브러리와 명령 줄 도구를 제공하는 컴퓨터 소프트웨어 프로젝트이다.
- elasticsearch의 데이터 핸들링은 cURL을 통하여 진행한다.
- 일래스틱서치는 기본값으로 자동으로 색인을 추가하고 타입을 위한 새로운 매핑도 생성한다.

## 2.4 데이터 검색하고 가져오기
- 데이터 검색하고 가져오기
![데이터 검색하고 가져오기](https://user-images.githubusercontent.com/49108738/69001768-9b2ccb00-0927-11ea-83eb-d4f6efdd39b2.png)
  - `/get-together/group`
    - get-together 색인의 group 타입에서 검색한다.
  - `q=elasticsearch`
    - "elasticsearch"를 포함한 문서를 찾는다.
  - `&fields=name,location`
    - 전체 문서에서 "elasticsearch"를 포함한 문서를 찾지만, 결과는 name과 location 필드만 반환한다. name과 location 필드에서 "elasticsearch" 를 찾는다는 의미가 아님을 주의하자.
  - `&size=1`
    - 검색결과가 몇 개 일지라도 `1`개의 결과문서만 반환한다.   
- 복수개의 타입에서 검색할 때는 `쉼표(,)`로 구분한다.
```bash
% curl "localhost:9200/get-together/group,event/_search\
?q=elasticsearch&pretty"
```
- 타입을 생략하면 전체 타입에서 검색한다.
```bash
% curl 'localhost:9200/get-together/_search?q=sample&pretty'
```
- 복수개의 인덱스에서 검색할 경우에도 `쉼표(,)`로 구분한다.
```bash
% curl "localhost:9200/get-together,other-index/_search\
?q=elasticsearch&pretty"
```
- 전체 인덱스에서 검색할 때는 인덱스를 생략하여 쿼리한다.
```bash
% curl 'localhost:9200/_search?q=elasticsearch&pretty'
```
- 응답내용
![응답내용](https://user-images.githubusercontent.com/49108738/69001974-2b204400-092b-11ea-98d5-e96cae12bb81.png)
- 적절한 쿼리 선택을 사용하여 검색하기
- 필터를 사용하여 검색하기
  - 필터 쿼리는 결과 데이터에 대하여 scoring 하지 않는다.
  - 다른 쿼리보다 빠르고 쉽게 캐시에 저장할 수 있다.
  - 점수가 없기 때문에 필터 결과는 점수에 의해 정렬되지 않는다.
- 집계 적용하여 검색하기
  - text 필드 데이터는 높은 카디널리티 필드를 로드 할 때 많은 힙 공간을 소비 할 수 있어서 `fielddata` 기본값이 `false`임.
  - agrregation 쿼리시에 `fielddata=true` 설정이 필요하다.[참조](https://www.elastic.co/guide/en/elasticsearch/reference/current/fielddata.html)
- ID로 문서를 가져올 수 있으며, 검색보다 훨씬 빠르고 자원 측면에서 저렴하다.~~elasticsearch를 사용하는 의미가...~~
- 문서가 존재하지 않으면 found feild의 value가 `false`다.

## 2.5 일래스틱서치 설정하기
```bash
brew info elasticsearch
```
- `elasticsearch.yml`에 클러스터 이름 명시하기 - 일래스틱 특유의 옵션이 들어가는 주 설정 파일이다.
  - homebrew 패키지를 통하여 설치한 경우 위치는 `/usr/local/etc/elasticsearch/elasticsearch.yml`
- `logging.yml`에 로깅 옵션 수정하기 - 로깅 설정 파일은 일래스틱서치가 로깅을 위해 사용하는 log4j의 옵션을 위한 것이다.
  - homrbrew 패키지를 통하여 설치한 경우 로깅 위치는 `/usr/local/var/log/elasticsearch`
  - 메인 로그(cluster-name.log) - 일래스틱서치가 동작 중일 때 무슨 일이 일어났는지에 관한 일반적인 정보를 알 수 있다. 예를 들어, 쿼리가 실패했거나 새로운 노드가 클러스터에 합류했는지 알 수 있다.
  - 느린 검색 로그(cluster-name_index_search_slowlog.log) - 쿼리가 너무 느리게 실행될 때 일래스틱서치가 로그를 남기는 곳이다. 기본으로 쿼리가 `0.5초` 넘게 걸리면 이곳에 로그를 남긴다.
  - 느린 색인 로그(cluster-name_index_indexing_slowlog.log) - 느린 검색 로그와 유사하지만 기본으로 색인 작업이 `0.5초` 이상 걸리면 로그를 남긴다.
  - logging 옵션변경 설정파일 logging.yml -> `log4j2.properties`
- 환경 변수나 `elasticsearch.in.sh`에 메모리 설정 조정하기 - 이 파일은 일래스틱서치를 작동시키는 자바 가상 머신(JVM)을 설정하기 위한 것이다.
  - 실행파일 : `/usr/local/Cellar/elasticsearch/6.8.4/bin/elasticsearch`

## 2.6 클러스터에 노드 추가하기
- 로컬에서 2개 이상의 노드를 시작하려면 elasticsearch.yml 파일의 적당한 위치에 설정값 `node.max_local_storage_nodes: 4`을 추가한다.
- 노드개수에 따라서 샤드의 활성화여부 및 분배를 확인할 수 있다.
  - 노드가 1개 일 때
![노드가 1개 일 때](https://user-images.githubusercontent.com/49108738/69058218-974b9680-0a56-11ea-8d60-e549358178fb.png)
  - 노드가 2개 일 때
![노드가 2개 일 때](https://user-images.githubusercontent.com/49108738/69058269-b2b6a180-0a56-11ea-8dbc-f69f76782196.png)
  - 노드가 3개 일 때 
![노드가 3개 일 때](https://user-images.githubusercontent.com/49108738/69058370-e09be600-0a56-11ea-95dd-7b29d2fc0d8a.png)

## Chpater03 preview
- 일래스틱서치에서 효율적으로 데이터를 조직한다
- 문서가 어떤 종류의 필드를 가질 수 있는지 배운다.
- 색인, 갱신, 삭제를 위한 모든 관련 옵션에 익숙해진다.
