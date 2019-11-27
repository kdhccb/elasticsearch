## Chapter03 데이터 색인, 변경, 삭제
- 같은 색인에서 도큐먼트의 다수 개 타입을 정의하는 매핍 타입 사용
- 매핑에 사용 가능한 필드 타입
- 사전 정의된 필드와 필드 옵션 사용
- 데이터 색인, 변경 및 삭제에 도움되는 모든 것
- 필드 유형
  - 기본 필드 : 문자열과 숫자
  - 배열 및 다중 필드 : 같은 필드에 같은 기본 타입으로 된 다수개의 값을 저장. 예를 들면, `tags` 필드는 다수개의 태그 문자열을 가질 수 있음
  - 사전 정의된 필드 : `_ttl`(Time to live), `_timestamp` 등

### 3.1 도큐먼트 종류를 정의하는 매핑 사용하기
- 논리적 분할 측면 : 색인(DB) > 타입(테이블) > 도큐먼트
![검색 어플리케이션](https://user-images.githubusercontent.com/49108738/69470628-2819cd80-0ddb-11ea-8f7e-134b138ffd15.png)
- 버전별 type history
  - 5.0 : started enforcing that fields that share the same name across multiple types have compatible mappings.
  - 6.0 : started preventing new indices from having more than one type and deprecated the \_default\_ mapping.
  - 7.0 : deprecated APIs that accept types, introduced new typeless APIs, and removed support for the \_default\_ mapping.
  - 8.0 : will remove APIs that accept types
- 기존 필드의 타입변경을 불가능하다. 해당 색인의 데이터를 모두 삭제하고 다시 색인한다.

### 3.2 도큐먼트 필드를 정의하는 기본 타입
- [data_type](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-data-types.html)
![data_type](https://user-images.githubusercontent.com/49108738/69471488-d70dd780-0de2-11ea-8df3-2e6e816fc07b.png)
- 문자열(text, keyword)
  - 기본 분석기는 모든 문자를 소문자로 만들고 문자열을 단어 단위로 분해한다.
![기본 분석기](https://user-images.githubusercontent.com/49108738/69471525-82b72780-0de3-11ea-86a7-6719c572cc00.png)
  - property의 index 기본값은 `analyzed`이며, 분석기는 모든 문자를 소문자로 바꾸고 단어 단위로 분해한다.
  - property의 index 값을 `not_analyzed`로 설정하면, 문석 과정을 건너 뛰고 전체 문자열은 단일 term으로 색인한다.
  - property의 index 값을 `no`로 설정하면, 색인을 생략하고 어떠한 term도 만들지 않아서 특정 필드의 검색이 불가능하다.
- 숫자
  - 숫자 타입은 기본 자바 기본 데이터 타입과 일치하며, 색인 크기와 색인할 값의 범위에 영향을 미친다.
  - 정밀도를 모르면 자동으로 정수는 `long`, 부동소수저 값으로는 `double`를 사용하도록 한다.
- 날짜
  - `2013-12-25T09:00:00`처럼 보통 문자열로 된 날짜를 제공하는데 사용하고, 문자열을 파싱하고 long 타입의 숫자로 루씬 색인에 저장한다.
  - 검색할 때, 날짜 문자열을 제공하지만 문자열을 파싱해서 백그라운드에서는 빠르게 저장하기 위해 숫자로 동작한다.
  - 기본적으로 ISO 8601(`2013-11-11T10:32:45.453-03:00`) 타임스탬프를 파싱한다.
- 불린
  - 불린 타입은 도큐먼트에 `true`나 `false` 값을 저장하기 위해 사용한다.
  
### 3.3 배열과 다중필드
- 배열
  - 다수 개 값으로 필드를 색인하려면, 대괄호(Square bracket`[]`)로 값을 둘러 싸 넣는다.
  - 같은 데이터로 더 많은 데이터를 색인한다.
- 다중 필드
  - 다중 필드는 같은 데이터를 다른 설정으로 여러번 색인한다.
  - 단일 필드를 다중필도로 re-index없이 upgrade 할 수 있다. 반대로는 가능하지 않은데, 한 번 지정하면 매핑에서 sub-field를 삭제할 수 없다.

## 3.4 사전 정의된 필드 사용하기
- 일반적으로, 사전 정의된 필드는 사용자가 정의하지 않고 일래스틱서치가 제공한다.
- 사전 정의된 필드는 필드에 따라 특수 기능을 갖고 있다. 예를 들면, `_ttl`필드는 특정 시간 이후에 도큐먼트를 자동 삭제하는 기능을 갖는다.
- 사전 정의된 필드 이름은 모두 언더스코어(`_`)로 시작한다.
- 도큐먼트를 저장하고 검색하는 방식 제어하기
  - 원래의 내용을 저장하는데 사용하는 `_source`
  - 모든 것을 색인하는 `_all`
    - 항상 특정 필드만 검색하려면 _all의 `enable` 옵션으로 `false`를 설정해서 비활성화할 수 있다.
    - 기본적으로 _all은 `include_in_all` 옵션 값으로 `true`를 가진 개별 필드를 암묵적으로 포함한다.
- 도큐먼트 식별하기
![\_id, \_type 필드 기본설정](https://user-images.githubusercontent.com/49108738/69472753-6d48fa00-0df1-11ea-8cd2-044f052e8d8d.png)
  - ID를 `1st` 이용해서 도큐먼트를 색인하려면 다음과 같이 명시적으로 색인한다.
![1st 색인](https://user-images.githubusercontent.com/49108738/69472822-260f3900-0df2-11ea-8944-5ca55bc7f51c.png)
  - 일래스틱서치가 ID를 생성하도록 하려면 HTTP `POST`를 사용한고 ID는 생략한다.
![ID 생략](https://user-images.githubusercontent.com/49108738/69472852-91590b00-0df2-11ea-999b-4212b5fd0aab.png)
  - 도큐먼트에 색인 이름을 저장하기 위해 ID 및 type과 마찬가지로 `_index` 필드를 사용한다.
  - 기본적으로 `_index` 값은 비활성이다.
  -\_index 값을 활성화하려면 매핑에서 `enable`옵션을 true로 설정한다.
![\_inedx 활성화](https://user-images.githubusercontent.com/49108738/69472912-5d321a00-0df3-11ea-878b-c0aade4e807b.png)

## 3.5 기존 도큐먼트 변경하기
![기존 도큐먼트 변경하기](https://user-images.githubusercontent.com/49108738/69478958-91c9c400-0e3b-11ea-862b-eac3a55493f6.png)
- 기존 도큐먼트를 조회 -> 명시된 변경사항을 반영 -> 기존 도큐먼트를 삭제하고 같은 자리에 변경사항이 반영된 새 도큐먼트를 색인
- 부분 도큐먼트 전송하기
  - 필드의 압력할 값을 가진 도큐먼트의 일부분을 전송한다. -> 도큐먼트의 URL의 \_update `endpoint`에 HTTP POST 요청으로 전송한다.
![update](https://user-images.githubusercontent.com/49108738/69479030-41069b00-0e3c-11ea-840f-13a98d0e6386.png)
  - `upsert`는 존재하지 않는 도큐먼트를 변경할 수 있다.
  - 스크립트를 이용해서 변경하기
    - 기본 스크립트 언어는 `Groovy`다.
    - \_source를 참조하려면 `ctx._source`를 사용하고 특정 필드를 참고하려면 `cst._source['필드 이름']` 또는 `cts._source.필드 이름`을 사용한다.
![스크립트를 이용해서 변경하기](https://user-images.githubusercontent.com/49108738/69494760-aa50e180-0f02-11ea-9aff-7c83c5f7735e.png)
- 버전 관리로 동시성 제어 구현하기
![버전 관리 도식화](https://user-images.githubusercontent.com/49108738/69494892-2992e500-0f04-11ea-8b2a-16dc19dc3459.png)
![버전 관리 코드](https://user-images.githubusercontent.com/49108738/69494897-33b4e380-0f04-11ea-83a8-e15406bf97da.png)
  - 버전 관리 코드 예제
    - 시점1 : 10초 이후에 update1 스크립트가 실행되어서 shert1에 대하여 price를 15 -> 2로 업데이트 한다. 이 때 버전은 1이다.
    - 시점2 : 10초 이내에 update2 스크립트가 실행되어서 shert1에 대하여 caption을 Learning ES -> Knowing ES로 업데이트 한다. 이 때 버전은 1이다.
    - 시점3 : update2 스크립트로 인하여 caption이 Knowing ES로 변경되고, Version이 1 -> 2로 변경되었다.
    - 시점4 : 시점1에서 확인했을 때 Version이 1 이었지만, 현재 Version2 로 확인되어 오류가 발생하고 update1이 실패한다.
  - 충돌 시, `retry_on_conflic` 옵션을 사용하면 자동으로 재시도 할 수 있다.
  - 충돌 예상시, 기대하는 버전이 3이라면 `version=3`이라는 옵션으로 재색인을 요청할 수 있다.

## 3.6 데이터 삭제하기
- 개별 도큐먼트 또는 도큐먼트 그룹 삭제 : 삭제할 대상을 검색시 보이지 않도록만 표시해서 나중에 비동기 방식으로 색인에서 삭제한다.
  - 단일 도큐먼트 삭제하기
    - % curl -XDELETE 'localhost:9200/online-shop/shirts/1'
  - 색인과 변경처럼 삭제도 동시성 관리를 위해 버전 관리를 사용할 수 있다.
  - 삭제 시, 60초 동안 버전을 유지하여 변경과 재생성에 대한 exception 처리를 할 수 있도록 한다.
  - 전체 매핑 타입까지 삭제한다. 매핑과 이 타입으로 색인된 모든 도큐먼트까지 삭제한다. 색인을 완전히 삭제하는 것에 비해 타입 삭제 시 더 많은 시간과 자원을 사용한다.
    - % curl -XDELETE 'localhost:9200/online-shop/shirts
  - get-together 색인으로 부터 "Elasticsearch"가 포함된 모든 도큐를 삭제하려면 다음 명령을 실행한다.
    - % curl -XDELETE 'localhost:9200/get-together/_query?q=elasticsearch'
- 완성된 색인 삭제 : 도큐먼트 그룹을 삭제하는 특수한 경우라서, 성능을 고려해서 사용할만한 방식은 아니다. 주요 작업은 그 색인과 연관된 모든 파일을 삭제하는 것인데, 거의 즉시 처리된다.
  - 색인 삭제하기
    - % curl -XDELETE 'localhost:9200/get-together/'
  - elasticsearch.yml에서 `destructive\_requires\_name` 옵션을 `true`로 변경하면, 삭제 시 일래스틱서치는 \_all뿐만 아니라 색인 이름으로 와일드카드(Wildcard)까지도 거부한다.
- 색인 닫기 : 닫은 색인은 읽기나 쓰기 작업을 허용하지 않고, 메모리에 load하지도 않는다. 디스크에 남아 있으며, 닫은 색인을 여는 작업만으로도 쉽게 복구할 수 있는 장점이 있다.
  - 색인 닫기
    - % curl -XPOST 'localhost:9200/online-shop/_close'
  - 색인 열기
    - % curl -XPOST 'localhost:9200/online-shop/_open'
