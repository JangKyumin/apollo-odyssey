# Odyssey Lift-off Study

Apollo Tutorial : https://www.apollographql.com/tutorials

## Apollo Tutorial Step 1

### 스키마 우선 설계

클라이언트 애플리케이션에 필요한 정확한 데이터를 기반으로 기능을 구현

#### 주요단계
1. 스키마 정의 : 기능에 필요한 데이터를 식별한 다음 가능한 한 직관적으로 해당 데이터를 제공하도록 스키마를 구성합니다.
2. 백엔드 구현 : 우리는 Apollo 서버를 사용하여 GraphQL API를 구축하고 필요한 데이터를 포함하는 데이터 소스에서 가져옵니다. 이 첫 번째 과정에서는 모의 데이터를 사용합니다. 다음 과정에서는 앱을 라이브 REST 데이터 소스에 연결합니다.
3. 프런트엔드 구현 : 클라이언트는 GraphQL API의 데이터를 사용하여 뷰를 렌더링합니다.

#### 이점

- 팀이 병렬로 작업할 수 있다.
- 전체 개발 시간을 단축한다.

### 데이터 그래프

앱의 데이터를 개체 (예: 학습 트랙 및 작성자) 의 컬렉션 및 개체 간의 관계 (예: 작성자가 있는 각 학습 트랙)로 본다.

개체를 Node로 생각 하고 각 관계를 두 노드 사이의 Edge로 생각하면 전체 데이터 모델을 노드와 가장자리의 그래프로 생각할 수 있다. 이것을 우리 애플리케이션의 그래프라고 한다.

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612409160/odyssey/lift-off-part1/LO_02_v2.00_04_53_09.Still002_g8xow6_bbgabz.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements]


#### 그래프 내용

- Node와 Edge의 모음이다.
- 앱의 데이터를 표현한다.

### GraphQL 스키마

Schema는 서버와 클라이언트 간의 계약과 같다. GraphQL API가 할 수 있는 것과 할 수 없는 것과 클라이언트가 데이터를 요청하거나 변경할 수 있는 방법을 정의한다. 백엔드 구현 세부 정보를 숨기면서 소비자에게 유연성을 제공하는 추상화 계층이다.

스키마의 핵심은 필드를 포함하는 개체 유형의 모음이다. 각 필드에는 고유한 유형이 있다. 필드 유형은 스칼라(예: Int 또는 String)이거나 다른 개체 유형일 수 있다.

type 키워드를 사용하여 유형을 선언한 후 유형 이름(PascalCase가 모범 사례)을 입력한 다음 괄호를 열어 포함된 필드를 보관한다.

필드는 이름(camelCase), 콜론, 필드 유형(스칼라 또는 개체)으로 선언한다.

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612409235/odyssey/lift-off-part1/type_spacecat_aymp3y_l04j48.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part1/schema-definition-language-sdl]


#### Descriptions

```
" ~~~ " : 한 줄로 주석 가능

""" 
    ~~~ : 여러 줄로 주석 가능
"""
```

### 스키마 정의

필드 유형 뒤에 느낌표가 붙는 이유는 null일 수 없다는 의미이다.

#### Query 타입

해당 타입의 필드는 다른 스키마에 대한 진입점이다. 클라이언트가 쿼리할 수 있는 최상위 필드를 의미한다.

``` javascript
type Query {
  tracksForHome: [Track!]!
}

type Track {
  id: ID!
  title: String!
  author: Author!
  thumbnail: String
  length: Int
  modulesCount: Int
}

type Author {
  id: ID!
  name: String!
  photo: String
}
```

### Apollo 서버

* 목적
    - 클라이언트로부터 들어오는 GraphQL 쿼리 수신
    - 새로 생성된 스키마에 대해 해당 쿼리의 유효성을 검사합니다.
    - 모의 데이터로 쿼리된 스키마 필드 채우기
    - 채워진 필드를 응답으로 반환

## Apollo Tutorial Step 2

### 데이트 흐름

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1617351987/odyssey/lift-off-part2/lop2-1-06_enfbis.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part2/journey-of-a-graphql-query]


#### Client

데이터를 가져오기 위해 GraphQL 서버로 쿼리를 전송합니다. 앱은 필요한 필드 선택 집합을 정의하는 문자열로 쿼리를 구성한다. 그런 다음 POST 또는 GET 요청으로 해당 쿼리를 서버로 보낸다.

#### Server

먼저 GraphQL 쿼리로 문자열을 추출한다. AST(Astract Syntax Tree)라고 불리는 트리 구조 문서로 변환한다. 이 AST를 사용하면 서버는 스키마의 유형 및 필드에 대해 쿼리를 검증한다.

요청된 필드가 스키마에 정의되어 있지 않거나 쿼리 형식이 잘못된 경우 서버는 오류를 발생시키고 이를 앱으로 바로 보낸다.

##### 역할

* 스키마에 대한 유효성 검사를 한다.
* 요청에 담긴 GraphQL 쿼리에 대한 문자열을 추출한다.
* GraphQL 쿼리 문자열을 추상 구문 트리로 변환한다.

##### 오류

* 요청받은 필드가 스키마에 정의되지 않으면 발생한다.
* 요청된 쿼리 문자열의 형식이 잘못되면 발생한다.

#### Resolver

쿼리의 각 필드에 대해 서버는 해당 필드의 해석기 기능을 호출한다. Resolver 함수의 임무는 데이터베이스 또는 REST API와 같은 소스애서 데이터를 필드에 맞춰 처리하는 것이다.


#### 최종 전송

HTTP 응답 본문의 data키에 할당하여 클라이언트로 전송한다.

### 데이터 탐색

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612408870/odyssey/lift-off-part2/lop2-2-01_actpy7.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part2/exploring-our-data]

#### 데이터 소스

리졸버가 검색하는 데이터는 데이터베이스, 타사 API, 웹후크 등 모든 종류의 위치에서 가져올 수 있다. 이를 데이터 소스라고 한다. GraphQL의 장점은 여러 데이터 소스를 혼합하여 클라이언트 앱의 요구 사항을 충족하는 API를 생성할 수 있다는 것이다.

해당 프로젝트에서는 Apollo에서 지원하는 REST API를 사용하여 데이터를 가져온다. 추후 MongoDB로 변경할 예정이다.

* 리졸버 함수는 데이트 소스에서 데이터를 검색한다.
* 단일 GraphQL API는 여러 데이터 소스와 연결이 가능하다.


### Apollo RESTDataSource

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612408870/odyssey/lift-off-part2/lop2-3-01_irdz0b.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part2/apollo-restdatasource]

REST API를 호출하기 위해 `fetch`를 사용 가능하다. 하지만, 만약 track이라는 api 호출 후 100개를 반환 받고 각 소유자를 확인하기 위해 100번 더 호출한다면 비효율적인 측면이 보이게된다.

이러한 문제를 해결하려면 REST API 호출에 대한 리소스 캐싱 및 중복 제거를 효율적으로 처리할 GraphQL용으로 특별히 설계된 RESTDataSource를 사용하면 된다.

#### resolver의 구성 요소

* parent
    - parent는 이 필드의 부모에 대한 리졸버의 반환 값입니다. 이것은 리졸버 체인을 다룰 때 유용할 것이다.
* args
    - args는 GraphQL 연산에 의해 필드에 제공된 모든 GraphQL 인수를 포함하는 개체이다. 특정 항목(예: 모든 트랙 대신 특정 트랙)을 쿼리할 때 클라이언트에서 서버의 이 args 매개 변수를 통해 액세스할 수 있는 id 인수로 쿼리를 만든다.
* context
    - context는 특정 작업에 대해 실행 중인 모든 확인자에서 공유되는 개체이다. 확인자는 인증 정보, 데이터베이스 연결 또는 RESTDataSource와 같은 상태를 공유하기 위해 이 컨텍스트 인수가 필요하다.
* info
    - info에는 필드 이름, 루트에서 필드로의 경로 등 작업 실행 상태에 대한 정보가 포함된다. 다른 항목처럼 자주 사용되지는 않지만 캐시 정책을 리졸버 수준으로 설정하는 등의 고급 작업에 유용할 수 있다.

* 필요없는 인자는 `_`로 구분하여 필요없는 인자부터 첫번째는 `_`를 두번째는 `__`를 사용한다.

## Apollo Tutorial Step 3

### Query 인수

#### 목적

* 사용자가 보낸 검색어 사용하기 위해
* 필드의 반환 값을 변환하기 위해 
* 특정 개체를 검색하기 위해
* 객체를 필터링하기 위해

### 리졸버 체인

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1623355358/odyssey/lift-off-part3/resolver-parent_kne6hn.jpg)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part3/resolver-chains]

리졸버 함수에는 네 가지 매개 변수가 있다. 첫 번째 부모는 해결사 체인에서 이전 함수의 반환된 데이터를 포함한다. 두 번째 args는 필드에 제공된 모든 인수를 포함하는 개체이다. 세 번째 매개 변수인 컨텍스트를 사용하여 데이터베이스 또는 REST API와 같은 데이터 소스에 액세스한다. 마지막으로, 네 번째 매개 변수인 info는 작업 상태에 대한 정보 속성을 포함한다.

```javascript
modules: ({id}, _, {dataSources}) => {
  return dataSources.trackAPI.getTrackModules(id);
},
```
위 코드를 보면 부모로부터 id 속성을 검색하기 위한 첫 번째 매개 변수를 재구성한다. args 매개 변수는 필요하지 않으므로 밑줄이 될 수 있으며 dataSources 속성의 세 번째 컨텍스트 매개 변수를 재구성할 수 있다.

내부에서 dataSources.trackAPI.getTrackModules 메서드를 호출하여 track의 ID를 전달한 결과를 반환할 수 있다.

## Apollo Tutorial Step 4

### Mutation

데이터를 수정하려면 다른 유형의 GraphQL 연산인 쓰기 연산인 돌연변이를 사용해야 한다.

Query 유형과 마찬가지로 Mutation 유형은 스키마의 진입점 역할을 한다. 우리가 지금까지 사용해온 스키마 정의 언어, 즉 SDL과 같은 구문을 따른다.

업데이트 작업의 특정 작업을 설명하는 동사로 add, delete, create를 붙이는 것이 좋다.

### Query vs Mutation

Query와 Mutation는 모두 GraphQL 작업의 유형이다. Query는 항상 데이터를 검색하는 읽기 작업입니다. Mutation는 항상 데이터를 수정하는 쓰기 작업입니다. Query 필드와 마찬가지로 Mutation 유형의 필드도 GraphQL API의 진입점입니다.

### Apollo 클라이언트 캐시

![](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1624653957/odyssey/lift-off-part4/doodle_client_cache_xzmvha.png)
[이미지 출처 : https://www.apollographql.com/tutorials/lift-off-part4/our-mutation-in-the-browser]

 Mutation API로 전송되는 동안에도 여전히 캐시에서 페이지를 로드한다. 성공적으로 반환되면 numberOfViews에 대한 업데이트된 값을 다시 받는다. Apollo Client는 이 track의 ID를 보고, 캐시에서 검색하고, numberOfViews 필드를 업데이트하는 작업을 배후에서 수행하고 있다. 캐시가 업데이트되면 UI도 업데이트된다.