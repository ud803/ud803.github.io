---
layout: post
title: <아랑고DB> 5. AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE
categories: 아랑고DB
tags: [아랑고DB, 그래프DB]

---
  
<div class="message">
저번 시간에 이어서 아랑고DB의 AQL을 활용해 데이터를 다루는 방법을 배워보자. RETURN, UPDATE 등의 문법을 살펴볼 것임
</div>

## 1. RETURN, UPDATE
### RETURN 
지난 시간에는 데이터를 삽입하는 `INSERT` 연산을 배워보았다. 이제 데이터를 넣었으니 꺼내보자.
AQL에서 데이터를 꺼내는 방법은 `RETURN` 연산을 통해서이다. 

<div class="exclamation">
아래 AQL 예시에서, RETURN 에서 하나의 AQL이 끝나기 때문에 한 줄씩 실행해야 한다!!
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

### UPDATE 
`UPDATE`는 컬렉션의 도큐먼트를 부분적으로 업데이트하는 연산이다. 
  
주어진 키로 도큐먼트를 찾아내어, 특정 필드를 업데이트한다.
  
<div class="tip">  
시스템 필드인 `_id`, `_key`, `_rev`는 업데이트가 불가능하고, 엣지 컬렉션의 `_from`과 `_to`는 업데이트가 가능하다는 점을 알아두자.
</div>
  
업데이트는 아래의 두 가지 문법 중 아무거나 사용해도 된다.
1. UPDATE _document_ IN _collection_
2. UPDATE _keyExpression_ WITH _document_ IN _collection_

{% highlight sql %}
// 첫번째 방식. 오브젝트 내에 _key값을 주면, 컬렉션 내의 해당 _key값을 매칭하여 나머지 값들을 업데이트한다.
// 아래 예시에서 _key가 178833인 도큐먼트를 찾아서, title을 Avengers 2001로 업데이트한다.
UPDATE {"_key": "178833", "title" : "Avengers 2001"} IN titles

// 두번째 방식. 위와 같은 결과를 만든다
UPDATE {"_key" : "178833"} WITH {"title" : "Avengers 2020"} IN titles
{% endhighlight %}
<div class="important">
UPDATE는 keyExpression을 통해 특정 _key를 기준으로 업데이트하기 때문에 다중 도큐먼트에 대한 업데이트를 신경 쓸 필요가 없다.
즉, _key는 항상 unique하기 때문에 UPDATE 연산도 고유하다.
</div>
두 방식은 언제 어떻게 사용하는게 편할까? 아래와 같은 상황에서는 두번째 방식이 더 편할 것이다.

상황 : _Avengers 2020 이라는 제목을 가진 영화를 Avengers 3030으로 업데이트하고싶다._
{% highlight sql %}
// 첫번째 방식
// new_doc 이라는 오브젝트를 새로 만들어주고, UPDATE 구문을 친다
FOR movie in titles
    FILTER movie.title == 'Avengers 2020'
    LET new_doc = {'_key' : movie._key, 'title' : 'Avengers 3030'}
    UPDATE new_doc IN titles
  
// 두번째 방식
// 바로 UPDATE 구문을 친다
FOR movie in titles
    FILTER movie.title == 'Avengers 2020'
    UPDATE movie WITH {'title' : 'Avengers 3030'} IN titles
{% endhighlight %}

필요할 때 상황에 맞게 쓰면 될 듯

  또한 `UPDATE` 구문은 업데이트 하기 전 도큐먼트와 하고 난 이후의 도큐먼트를 반환할 수 있다. 
{% highlight sql %}
UPDATE document IN collection options RETURN OLD
UPDATE document IN collection options RETURN NEW
UPDATE keyExpression WITH document IN collection options RETURN OLD
UPDATE keyExpression WITH document IN collection options RETURN NEW
{% endhighlight %}
  
`UPDATE`도 `INSERT`와 마찬가지로 여러가지 옵션을 줄 수 있다. 
- `ignoreErrors` : 업데이트에서 발생하는 에러 (unique key constraint violation이나 non-existing document) 무시
- `keepNull` : `null`로 필드 업데이트 할 수 있게 해줌. 디폴트로는 `null`이 저장이 안 된다. (필드가 안생김)
- `mergeObjects` : 오브젝트 필드의 경우, 업데이트 오브젝트에 명시되지 않은 필드들을 `merge`해서 그대로 둘 지, 아니면 없앨지를 결정한다.
 
마지막 옵션만 조금 더 설명하자면, {'target' : {'a' : 1, 'b': 2}}인 도큐먼트에서 {'target' : {'a': 3}}으로 업데이트 하는 상황을 가정하면,
- `mergeObjects`가 True인 디폴트의 경우, {'target' : {'a':3, 'b': 2}}가 된다
- False로 지정하면, 업데이트 대상이 아닌 필드는 날아간다. {'target' : {'a':3}}이 된다.
  
  
## 2. 어디까지 왔나
2편까지 기초 연산을 끝내려고 했는데, 생각보다 길어지고 있다. 빠르게 대충 치고 넘어가려다가 아랑고DB에 관한 손쉬운 한글 튜토리얼을 만들겠다는 다짐을 되살렸다. 조금 내용이 많아지더라도 각 부분에서 다룰 수 있는 부분은 다 다루려고 한다.  
3편에서 기초가 끝날지 모르겠지만, 다음 글에서는 REPLACE / UPSERT / REMOVE 내용을 다룬다!

  
1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. [AQL(Arango Query Lang) 배워보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)
5. **(지금 보고있는 글) AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE**
6. [AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/14/ArangoDB-6-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-3/)
7. [그래프 개념잡기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/23/ArangoDB-7-%EA%B7%B8%EB%9E%98%ED%94%84-%EA%B0%9C%EB%85%90-%EC%9E%A1%EA%B8%B0/)
8. [그래프 횡단하기 Graph Traversal](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/12/05/ArangoDB-8-%EA%B7%B8%EB%9E%98%ED%94%84-%ED%9A%A1%EB%8B%A8%ED%95%98%EA%B8%B0-Graph-Traversal/)
9. [데이터 모으기 COLLECT / AGGREGATE / MIN_BY, MAX_BY](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/12/11/ArangoDB-9-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%AA%A8%EC%9C%BC%EA%B8%B0-COLLECT-AGGREGATE-MIN_BY-MAX_BY/)
