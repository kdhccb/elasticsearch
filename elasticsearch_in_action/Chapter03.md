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
- 논리적 분한 측면 : 색인(DB) > 타입(테이블) > 도큐먼트
![검색 어플리케이션](https://user-images.githubusercontent.com/49108738/69470628-2819cd80-0ddb-11ea-8f7e-134b138ffd15.png)
- 버전별 type history
  - 5.0 : started enforcing that fields that share the same name across multiple types have compatible mappings.
  - 6.0 : started preventing new indices from having more than one type and deprecated the \_default\_ mapping.
  - 7.0 : deprecated APIs that accept types, introduced new typeless APIs, and removed support for the \_default\_ mapping.
  - 8.0 : will remove APIs that accept types
