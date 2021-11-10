---
layout: post
title: <ArangoDB> 5. AQL(Arango Query Lang) 배워보기 2 - Read / Update / Delete
categories: ArangoDB
---
  
<div class="message">
저번 시간에 이어서 아랑고DB의 AQL을 활용해 데이터를 다루는 방법을 배워보자. READ, UPDATE, DELETE, UPSERT 등의 문법을 살펴볼 것임
</div>

## 1. READ, UPDATE, DELETE, UPSERT 
### READ 
지난 시간에는 데이터를 삽입하는 `INSERT` 연산을 배워보았다. 이제 데이터를 넣었으니 꺼내보자.
AQL에서 데이터를 꺼내는 방법은 `RETURN` 연산을 통해서이다. 

<div class="exclamation">
아래 AQL 예시들에서 RETURN 구문에서 문법이 끝나기 때문에 한 줄씩 실행해야 함!!
</div>
  
{% highlight sql %}
// 무언가를 Return 하려면 아래처럼 쓴다
RETURN _expression_

// 예를들어, 문자열 hello world를 리턴한다
RETURN 'hello world'
{% endhighlight %}

위 문자열을 리턴했는데 이상한 점이 있다. 단순히 'hello world'가 아니라 ['hello world']로 배열이 리턴되었기 때문이다.
이는 AQL이 쿼리의 결과값을 항상 배열로 리턴하기 때문이다. 궁금하면 [여기](https://www.arangodb.com/docs/stable/aql/fundamentals-query-results.html) 읽어보셈
  
이제 컬렉션의 도큐먼트에 접근해보자. [컬렉션은 도큐먼트의 배열로 취급](https://www.arangodb.com/docs/stable/aql/fundamentals-document-data.html)되기 때문에, `FOR` 반복문을 통해 접근한다.
{% highlight sql %}
// 도큐먼트의 _id 값을 조회하여 직접 불러온다. 가장 빠르게 데이터에 접근하는 방법이다.
RETURN DOCUMENT('titles/178829')

// 컬렉션의 데이터를 조회하기
// 프로그래밍 언어의 반복문처럼 앞의 title은 사용자가 지정하는 임의의 변수다
FOR title in titles
    RETURN title

// 위 AQL과 똑같음
FOR abcabc in titles
    RETURN abcabc
{% endhighlight %}

이번에는 전체 도큐먼트가 아닌, 영화의 제목만을 리턴해보자
`RETURN DISTINCT`의 쓰임도 알아두자
  
{% highlight sql %}
// 영화의 제목이 title이라는 이름을 가지고 있어서, 혼동을 줄이기 위해 반복 변수 이름을 movie로 했음
// 이래서 네이밍이 중요하다
FOR movie in titles
    RETURN movie.title
  
// 지금은 titles 컬렉션의 title 필드에 unique constraint이 있기 때문에 중복되는 타이틀이 없다
// 만약 제약 조건이 없는데 중복을 거르고 리턴하고 싶다면?
FOR movie in titles
    RETURN DISTINCT movie.title
{% endhighlight %}

이번에는 [연산자(Operator)](https://www.arangodb.com/docs/stable/aql/operators.html)를 사용해보자.
  
조건을 걸 때는 `FILTER`와 함께 사용한다.
{% highlight sql %}
// 어벤져스의 50번째 이상 시리즈만 보고싶다면?
FOR movie in titles
    FILTER movie.title >= 'Avengers 50'
    RETURN movie
 
// 어벤져스 100번째 시리즈를 리턴하고 싶다면?
FOR movie in titles
    FILTER movie.title == 'Avengers 100'
    RETURN movie
{% endhighlight %}
위 예시에서 >= 'Avengers 50' 와 같은 표현은 사실 위험하다. 앞의 'Avengers'가 무조건 똑같이 위치한다고 가정하고, 숫자로만 크기를 비교하는 것이기 때문
'The Avengers 1' 처럼 The가 붙으면 당연히 얘가 'Avengers 50'보다 더 커진다. 예시를 위해 대충 한거니까 실제 사용할 때는 주의하자.
  
마지막으로 한 가지 재미있는 예시 하나만 보고가자. 각각의 도큐먼트는 서로 다른 필드를 가지고 있다. 만약 도큐먼트에 없는 필드로 조건을 걸면 어떻게 될까?
{% highlight sql %}
// 시청 연령이 19세 이하인 영화만 리턴하고 싶다
// 그런데 age 라는 필드는 만든 적도 없다
FOR movie in titles
    FILTER movie.age <= 19
    RETURN movie
{% endhighlight %}

위 쿼리를 날리면 놀랍게도 컬렉션의 모든 도큐먼트가 리턴된다. 이는 존재하지 않는 데이터를 참조하게 되면, 해당 필드는 `null`로 취급되는데, `null`은 데이터 타입 사이에서 [순서상 가장 작기 때문이다.](https://www.arangodb.com/docs/stable/aql/fundamentals-type-value-order.html)
따라서 존재하지 않는 `movie.age`는 `null`이고, 이는 19보다 작으므로 True가 되어 모든 데이터가 리턴된다. 

원래 의도에 맞게 다시 AQL을 구성해보면, 아래처럼 하면 된다.
{% highlight sql %}
// HAS 연산은 해당 오브젝트에 age라는 필드가 있는지 확인한다
FOR movie in titles
    FILTER HAS(movie, 'age')
    FILTER movie.age <= 19
    RETURN movie    
{% endhighlight %}

### Update 

  
## 4. 어디까지 왔나
생각보다 쓸 내용이 많아져서 AQL 기초 관련 내용을 1편과 2편으로 나누려고 한다. 2편은 도큐먼트를 삽입하는 또다른 방법인 UPSERT를 알아보고, 나머지 Read, Update, Delete 를 간단히 짚고 넘어가려고 한다. 그리고 그래프 횡단은 내용이 방대하기에 따로 분리할 예정이다.

  
1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/arangodb/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/arangodb/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/arangodb/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. **(지금 보고있는 글) AQL(Arango Query Lang) 배워보기 1**
5. AQL(Arango Query Lang) 배워보기 2 - Read / Update / Delete
6. AQL(Arango Query Lang) 배워보기 3 - Graph Traversal
7. [Python-Arango](https://github.com/ArangoDB-Community/python-arango) 라이브러리 활용하여 기본 기능 익히기
8. Relational/Document to ArangoDB Mapper 써서 데이터 대량으로 넣어보기
9. 넣은 데이터로 간단한 실습해보기
10. 넣은 데이터로 깊이있는 실습해보기
