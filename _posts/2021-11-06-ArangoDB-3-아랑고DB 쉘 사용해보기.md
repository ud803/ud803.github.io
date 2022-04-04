---
layout: post
title: <아랑고DB> 3. 아랑고DB 사용해보기 
categories: 아랑고DB
tags: [아랑고DB, 그래프DB]
---
  
<div class="message">
이번 글에서는 아랑고DB의 쉘을 사용해 아랑고DB의 기초 사용법을 배워보려고 한다. 사실 실제 작업은 Python 커넥터를 통해서 하는 경우가 대부분이지만, 실체를 알기 위해서는 직접 쉘로 붙어보는 것이 가장 효과적이라고 생각하기 때문이다.
</div>

## 들어가기에 앞서
  - 아랑고DB가 이미 설치되어있다고 가정한다. 만약 아직 설치되지 않았다면 [여기](https://ud803.github.io/arangodb/2021/11/02/ArangoDB-2/)를 보고 따라 설치해보자.
  - 또한 아랑고쉘은 JavaScript Shell 환경을 사용한다고 한다. 따라서 JS문법을 그대로 이용할 수 있다.
  
## 1. 데이터베이스 수준 명령어
  초기 실행은 `arangosh` 명령어를 통해 한다. 그리고 초기에 입력했던 `root` 계정의 비밀번호를 입력해주면 된다. 
  쉘에 접속 후 `tutorial` 명령어를 입력하면 간단한 튜토리얼도 따라할 수 있으니 참고하자.
  
  쉘에 접속했을 때 기본 데이터베이스는 `_system` 이다. 이제 `movies`라는 데이터베이스를 만들고, `titles`라는 도큐먼트 컬렉션을 만들어 볼 것이다.
  데이터베이스 이름을 짓는 [Naming Convention](https://www.arangodb.com/docs/stable/data-modeling-naming-conventions-database-names.html)도 읽어보자.
  
{% highlight js %}
// 현재 아랑고DB에 있는 데이터베이스들의 목록을 조회한다 
127.0.0.1:8529@_system> db._databases()
[ 
  "_system" 
]

// movies라는 데이터베이스를 만든다
127.0.0.1:8529@_system> db._createDatabase('movies')
true

// 생성된 것을 확인할 수 있다
127.0.0.1:8529@_system> db._databases()
[ 
  "_system", 
  "movies" 
]

// 데이터베이스를 변경해준다
127.0.0.1:8529@_system> db._useDatabase('movies')
true

// 컬렉션을 생성해준다. 탭을 눌러 자동완성 기능을 애용하자
// 컬렉션은 Edge Collection과 Document Collection으로 나뉘는데, 여기서는 단순히 도큐먼트의 집합을 생성할 예정이다.
127.0.0.1:8529@movies> db._createDocumentCollection('titles')
[ArangoCollection 139708, "titles" (type document, status loaded)]

127.0.0.1:8529@movies> db._collections()
[ 
  [ArangoCollection 139603, "_analyzers" (type document, status loaded)], 
  [ArangoCollection 139618, "_appbundles" (type document, status loaded)], 
  [ArangoCollection 139615, "_apps" (type document, status loaded)], 
  [ArangoCollection 139606, "_aqlfunctions" (type document, status loaded)], 
  [ArangoCollection 139621, "_frontend" (type document, status loaded)], 
  [ArangoCollection 139600, "_graphs" (type document, status loaded)], 
  [ArangoCollection 139612, "_jobs" (type document, status loaded)], 
  [ArangoCollection 139609, "_queues" (type document, status loaded)], 
  [ArangoCollection 139708, "titles" (type document, status loaded)] 
]
{% endhighlight %}

도큐먼트 컬렉션과 엣지 컬렉션은 데이터 모델의 성격에 따라 구분되는 컬렉션이다. 도큐먼트 컬렉션은 그래프DB에서 노드(버텍스)의 개념이고, 엣지 컬렉션은 이 노드들 사이의 관계를 나타내는 컬렉션이다.

## 2. 컬렉션 수준 명령어
이제 생성된 컬렉션 안에 도큐먼트를 추가해보자. 사실 쉘에서 데이터를 다루는 방식은 불편하기 때문에 이런 방법도 있다 정도로만 끝내려고 한다.
컬렉션 수준에서 사용하는 명령어들은 `db.컬렉션명`의 형태에 CRUD 관련 명령어들이 붙는 형태이다.
예를 들어 다음과 같은 식이다.
  - Create : db.titles.save({name : "Harry"})
  - Read : db.titles.toArray()
  - Update : db.titles.update('140292', {name : "Sally"})
  - Delete : db.title.remove('140292')
  
### CREATE 
`save` 명령어는 도큐먼트를 저장하기 위해 쓰이는데, 모든 데이터의 검색에서 쓰이는 `_key` 값을 직접 설정할 수 있다.
- `_key`는 컬렉션 레벨에서 **고유한** 값이다.
- `_id`는 데이터베이스 레벨에서 **고유한** 값이며, '{컬렉션명}/{_key 값}' 의 형태이다.
- 자세한 내용은 [여기](https://www.arangodb.com/docs/stable/appendix-glossary.html#document-revision)를 읽어보기
  
`_key`값을 어떻게 설정하느냐에 따라 쿼리의 효율이 하늘과 땅 차이이기 때문에 설계를 잘 해보자. 키 이름의 [네이밍 컨벤션](https://www.arangodb.com/docs/stable/data-modeling-naming-conventions-document-keys.html)도 확인해보자.
 
 {% highlight js %}
// Harry Potter 1이라는 도큐먼트를 생성한다. _key값을 지정해주지 않으면 자동으로 생성해준다.
127.0.0.1:8529@movies> db.titles.save({title: "Harry Potter 1"})
{ 
  "_id" : "titles/140292", 
  "_key" : "140292", 
  "_rev" : "_dNTU1bG---" 
}

// JS 문법을 활용해 반복문도 쓸 수 있다.
127.0.0.1:8529@movies> for (var i=0; i< 10; ++i) {
...> db.titles.save({name: "Harry Potter" + i});
...> }
{ 
  "_id" : "titles/141195", 
  "_key" : "141195", 
  "_rev" : "_dNT028W--A" 
}
{% endhighlight %}

### READ
데이터를 조회하는 방법도 간단하다.
  - `toArray()`를 통해 해당 컬렉션의 전체 도큐먼트를 조회하거나
  - `byExample({조건})`의 형태로 조건에 해당하는 도큐먼트만 조회할 수 있다
  
 {% highlight js %}

// toArray() 함수를 통해 컬렉션의 모든 도큐먼트를 조회한다
127.0.0.1:8529@movies> db.titles.toArray()
[ 
  { 
    "_key" : "140292", 
    "_id" : "titles/140292", 
    "_rev" : "_dNTU1bG---", 
    "title" : "Harry Potter 1" 
  } 
]

// 위에서 _key값은 설정하지 않았지만, 직접 도큐먼트를 조회하기 위해서는 document 함수에서 _key값을 불러주면 된다.
// 이렇게 _key값으로 도큐먼트를 조회하는 방법이 도큐먼트를 조회하는 가장 빠른 방법임을 새겨두자
127.0.0.1:8529@movies> db.titles.document('140292')
{ 
  "_key" : "140292", 
  "_id" : "titles/140292", 
  "_rev" : "_dNTU1bG---", 
  "title" : "Harry Potter 1" 
}

// 컬렉션 이름을 문자화하여 호출할 수도 있다
127.0.0.1:8529@movies> db._document("titles/140292")
{ 
  "_key" : "140292", 
  "_id" : "titles/140292", 
  "_rev" : "_dNTU1bG---", 
  "title" : "Harry Potter 1" 
}
 
// 이름이 Harry Potter8인 도큐먼트 조회
127.0.0.1:8529@movies> db.titles.byExample({name : "Harry Potter8"}).toArray();
[ 
  { 
    "_key" : "141193", 
    "_id" : "titles/141193", 
    "_rev" : "_dNT028W--_", 
    "name" : "Harry Potter8" 
  } 
]

{% endhighlight %}

### UPDATE
도큐먼트 업데이트는 `update(업데이트 매치 조건, 업데이트 할 필드)` 형태로 쓴다. 여기서 주목할 점은 리턴값으로 `_rev`, `_oldRev`를 제공한다는 점이다. 
rev는 revision의 약자로 일종의 도큐먼트의 버전과 같은 개념인데, 아랑고쉘의 tutorial에서는 이 revision 이 조건부 수정을 위한 용도로 쓰인다고 말한다. 즉, 도큐먼트를 동시에 업데이트 하는 동시성 문제가 발생했을 때 **상황에 따라서 revision 개념을 사용하면 의도하지 않았던 결과를 막을 수 있다**는 개념인 것 같다. 
 
 {% highlight js %}
// revision 설명 전에 잠깐 보면, 업데이트를 하면 기존의 old revision과 새롭게 바뀐 revision을 리턴해준다
// 즉, 도큐먼트의 버전이 바뀌었다고 안내해준다
127.0.0.1:8529@movies> db.titles.update('140292', {series : 1})
{ 
  "_id" : "titles/140292", 
  "_key" : "140292", 
  "_rev" : "_dNTcxl6---", 
  "_oldRev" : "_dNTU1bG---" 
}
  
// 여기서부터는 아랑고쉘에서 쉽게 설명해주는 부분이다
// 우선 도큐먼트를 조회해 doc이라는 변수에 저장한다
 > doc = db._document("places/example1");
// 그리고 아래 업데이트를 수행하면, 이는 별 문제없이 성공한다. 이때, 업데이트가 성공했기 때문에 기존의 revision 값은 변경됐다 
> db._update("places/example1", { someValue: 23 });
// 아래 업데이트는 위에서 선언한 doc을 기준으로 한다. 하지만, doc의 revision은 이미 위 업데이트 이후 old revision이 되었기 때문에 업데이트는 실패하게 된다. 
> db._update(doc, { someValue: 42 });
JavaScript exception in file '/usr/share/arangodb3/js/client/modules/@arangodb/arangosh.js' at 99,7: ArangoError 1200: conflict, _rev values do not match
!      throw error;
!      ^
stacktrace: ArangoError: conflict, _rev values do not match
{% endhighlight %}

### DELETE
도큐먼트를 삭제하는 것도 매우 간단하다. 마찬가지로 도큐먼트의 `_id`나 `_key` 값을 `remove()` 함수에 전달하면 된다.
{% highlight js %}
// _id로 지우기
127.0.0.1:8529@movies> db._remove("titles/140292");
{ 
  "_id" : "titles/140292", 
  "_key" : "140292", 
  "_rev" : "_dNTpPd----" 
}
  
// _key로 지우기
127.0.0.1:8529@movies> db.titles.remove("140292");
  
{% endhighlight %}

## 3. AQL 실행하기
AQL은 Arango Query Lang의 약자로, SQL처럼 편리하게 아랑고DB를 조회하고, 횡단하기 위한 언어이다. 아랑고쉘에서도 AQL을 실행할 수 있으며, 어떻게 하는지만 언급하고 나중에 제대로 배워보기로 한다.
  
{% highlight js %}
// 쿼리는 멀티라인이기 때문에 키보드 숫자 1 옆의 `(backtick이라고 부름) 로 쿼리를 감싸주도록 한다
// 아래 AQL은 titles 컬렉션에 있는 모든 도큐먼트를 리턴하는 쿼리다
// db.titles.toArray()와 동일하다
127.0.0.1:8529@movies> db._query(`
...> FOR title in titles
...> RETURN title`)
{% endhighlight %}

## 4. 어디까지 왔나
다음 글에서는 아랑고DB의 Web UI를 통해 우리가 넣은 데이터를 살펴보고, 다양한 기능들을 살펴보려고 한다.

1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. **(지금 보고있는 글)아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기**
4. [AQL(Arango Query Lang) 배워보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)
5. [AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)
6. [AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/14/ArangoDB-6-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-3/)
7. [그래프 개념잡기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/23/ArangoDB-7-%EA%B7%B8%EB%9E%98%ED%94%84-%EA%B0%9C%EB%85%90-%EC%9E%A1%EA%B8%B0/)
8. [그래프 횡단하기 Graph Traversal](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/12/05/ArangoDB-8-%EA%B7%B8%EB%9E%98%ED%94%84-%ED%9A%A1%EB%8B%A8%ED%95%98%EA%B8%B0-Graph-Traversal/)
9. [데이터 모으기 COLLECT / AGGREGATE / MIN_BY, MAX_BY](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/12/11/ArangoDB-9-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%AA%A8%EC%9C%BC%EA%B8%B0-COLLECT-AGGREGATE-MIN_BY-MAX_BY/)
10. [프로젝트. 그래프를 통한 영화 추천시스템 만들어보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2022/01/16/ArangoDB-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8.-%EA%B7%B8%EB%9E%98%ED%94%84%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%98%81%ED%99%94-%EC%B6%94%EC%B2%9C%EC%8B%9C%EC%8A%A4%ED%85%9C-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0-1/)
11. [프로젝트. 그래프를 통한 영화 추천시스템 만들어보기 2 (최종편)](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2022/04/05/ArangoDB-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8.-%EA%B7%B8%EB%9E%98%ED%94%84%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%98%81%ED%99%94-%EC%B6%94%EC%B2%9C%EC%8B%9C%EC%8A%A4%ED%85%9C-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EA%B8%B0-2-(%EC%B5%9C%EC%A2%85%ED%8E%B8)/)
