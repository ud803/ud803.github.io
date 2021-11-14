---
layout: post
title: <ArangoDB> 6. AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE
tags: [아랑고DB, 그래프DB]
---
  
<div class="message">
데이터를 조작하는 AQL에 대해 공부하는 마지막 시간이다. REPLACE, UPSERT, REMOVE 등의 연산을 소개하고, 이때까지 AQL1~3 시리즈에서 배운 연산들을 정리하는 시간을 가지려고 한다.
</div>

## 1. REPLACE, UPSERT, REMOVE
### REPLACE
`UPDATE`가 `_key`값이 일치하는 도큐먼트의 일부분을 바꾸었다면 `REPLACE`는 도큐먼트 자체를 바꾸는 연산을 한다.

<div class="tip">
UPDATE와 마찬가지로 시스템 필드인 `_id`, `_key`, `_rev`는 업데이트가 불가능하고, 엣지 컬렉션의 `_from`과 `_to`는 업데이트가 가능하다.
</div>

문법도 `UPDATE`와 동일하다. `_key`값이 포함된 도큐먼트를 전달해 한 번에 바꾸거나, `_key`값을 먼저 지정해주고, 이후에 도큐먼트를 전달하는 방식이다.

1. REPLACE _document_ IN collection
2. REPLACE _keyExpression_ WITH document IN collection

나머지 방식이나 Options 같은 경우도 `UPDATE`와 동일하기 때문에 건너뛴다. `UPDATE`에 관한 이전 글은 [여기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)를 보면 됨

### REMOVE
`UPSERT` 전에 쉬운 `REMOVE` 먼저 잠깐 살펴본다. 이름 그대로 도큐먼트를 제거하는 연산이며, _keyExpression_ 을 사용한다. 즉, `_key`값을 기준으로 제거할 도큐먼트를 찾는다.

아래 문법처럼 사용하며, 간단하니까 얘도 생략한다.

- REMOVE _keyExpression_ IN _collection_


### UPSERT
마지막으로 볼 연산자는 `UPSERT`라는 재미있는 연산이다. `UPSERT`는 `UPDATE`와 `INSERT`의 합성어로써 아래 두 가지 상황에 맞는 동작을 한다.
- 데이터가 없으면 `INSERT`를 한다.
- 데이터가 이미 존재하면 `UPDATE`나 `REPLACE`를 한다.

문법은 아래 두 가지 형태를 취한다.
1. UPSERT _searchExpression_ INSERT _insertExpression_ UPDATE _updateExpression_ IN collection
2. UPSERT _searchExpression_ INSERT _insertExpression_ REPLACE _updateExpression_ IN collection

여기서 주목해야할 점은 _keyExpression_ 을 통해 `_key`를 기준으로하는 `UPDATE` 연산과 달리, `UPSERT`는 _searchExpression_ 을 기준으로 삼는다는 점이다. 이는 아래 세 가지 포인트를 갖는다.
  
- **`_key`값에 국한되지 않고 여러 필드로 조건을 걸 수 있다!**
- **동시에, `_key`는 고유했지만 필드는 고유하지 않을 수 있기에 여러 도큐먼트가 `UPSERT`의 대상이 될 수 있다**
- **그리고 그들 중 하나가 임의로 대상 지정된다**
  
<div class="important">
따라서 UPSERT의 lookup 과정에서 성능을 높이기 위해서는 반드시 lookup 대상의 필드들에 인덱스를 걸어주어야 한다. 그렇지 않으면 UPSERT의 성능은 현저하게 떨어진다.
</div>

이제 실제 연산을 실행해보자. 컬렉션에 관한 설정은 [여기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)에서 `CREATE` 부분의 노란 박스를 보면 됨.

`titles` 컬렉션의 인덱스 설정은 그대로 둔 채, 데이터만 `truncate` 해주자.

{% highlight sql %}
// 빈 컬렉션에 해리포터 시리즈를 7편 넣어준다
FOR i in 1..7
  INSERT {
      'title': CONCAT('Harry Potter ', i), 
      'series': 'Harry Potter'
  } INTO titles
{% endhighlight %}

{% highlight sql %}
// 이제 해리포터 시리즈에 UPSERT 연산을 해보자
// 임의의 해리포터 도큐먼트(총 7개) 중 하나에 `series_num`이라는 필드가 추가되었다
UPSERT {'series': 'Harry Potter'}
  INSERT {'meaning': 'less'}
  UPDATE {'series_num': 1}
INTO titles
{% endhighlight %}

{% highlight sql %}
// 이번에는 없는 도큐먼트에 대해 UPSERT를 해보자
UPSERT {'series': 'Avengers'}
  INSERT {'meaning': 'less'}
  UPDATE {'series_num': 1}
INTO titles
{% endhighlight %}

위 연산은 대상 도큐먼트가 없기 때문에 `INSERT` 연산을 수행하게 되어 새로운 도큐먼트를 만든다.

그럼 위 연산을 한 번 더 실행하면 어떻게 될까? 지금 생각으로는 또다른 {'meaning': 'less'} 도큐먼트를 삽입할 것 같다.

직접 해보자

{% highlight sql %}
// 원모어타임
UPSERT {'series': 'Avengers'}
  INSERT {'meaning': 'less'}
  UPDATE {'series_num': 1}
INTO titles
{% endhighlight %}

`title` 필드에 고유 인덱스 설정을 잘 주었다면, 위 AQL은 오류를 일으킨다. `title`은 반드시 unique해야 하는데, {'meaning': 'less'} 도큐먼트는 'title'을 `null`값으로 가지고 있는 도큐먼트이기 때문이다.

따라서 해당 도큐먼트를 2개 넣으면 `null`이 두 개가 되어 고유값 에러를 일으킨다. 재미있는 발견이다.

<div class="warning">
UPSERT 연산에서 lookup 파트와 insert/update/replace 파트는 아토믹하지 않게 실행이 된다. 즉, 동시에 여러 UPSERT 쿼리를 쳤을 경우 lookup 과정에서 도큐먼트가 존재하지 않다고 인식하고 모든 쿼리가 INSERT 연산을 수행할 수 있다는 것이다.

따라서 찾으려는 lookup 키 (searchExpression)에는 unique constraint을 주는 것이 낫다. 이는 몽고DB의 UPSERT에서도 동일하게 발생하는 문제이다. 잘 알아두자.
</div>

<div class="warning">
UPSERT를 대량으로 하게되면 intermediate commit을 하게 되어 전체 연산이 끝나기 전에 중간 연산을 write하게 된다!
</div>

`UPSERT`는 매우 유용한 기능이지만, 위와 같은 한계점들이 존재한다. 또한, `UPSERT`는 아랑고DB에서 HTTP API를 제공해주지 않는 연산 중 하나이다. 나머지 연산들은 모두 API가 열려있지만, `UPSERT`만은 AQL을 통해서만 수행이 가능하다.

이는 많은 아랑고 유저들이 개선을 원하는 부분 중 하나인데 아직까지 해결되고 있지는 않다. [깃헙 이슈](https://github.com/arangodb/arangodb/issues/2542) 참고.

## 2. CRUD Best Practice

AQL 기초 1~3편을 통해 기초 연산들을 배워보았다. 그럼 언제 어느 연산을 사용해야 할까?

데이터를 넣고, 업데이트하는 몇가지 상황을 생각해보자.

<div class="tip">
정말 많은 데이터를 관리해야 한다면, HTTP API를 통해 직접 ArangoDB에 데이터를 보내는 게 가장 빠른 방법이다.
다른 언어의 라이브러리를 통해서 하면 편리하긴 하지만, 효율이 최우선이라면 HTTP API를 살펴보자.
</div>

### 데이터를 넣을건데, (영화명, 영화시리즈번호)의 쌍이 고유했으면 좋겠다.

이 경우는 이때까지 예제로 살펴봤던 것과 약간 다르게, 여러 개의 필드쌍이 하나의 `_key`를 구성해야 하는 경우이다. 

여러 방법이 있는데, 1) index에 컴마로 분리하여 두 개 필드를 넣어 unique 제한을 주거나 2) 애초에 `_key`값을 `영화명_영화시리즈번호`로 만들어 넣는 것이다.

나는 후자의 방법이 가장 빠른 시스템 인덱스인 `_key`를 사용하고, 별도의 추가 인덱스를 생성하지 않기 때문에 선호하는 편이다.

이렇게 인덱스를 설정한 후, `INSERT`로 `ignoreErrors:true`, `overWrite:ignore` 를 줘서 빠르게 밀어넣는다.


### 시간별 통계 데이터를 넣을건데, 10분마다 업데이트되는 값을 반영하고싶다.

예를 들어, 시간별 블로그 이용자 통계를 만들고 싶다고 가정해보자. 이 통계의 업데이트 주기가 10분이라면 15시 데이터에 대해, 15시 10분에 측정한 값과 15시 20분에 측정한 값은 다를 것이다. (10분동안 사람들이 더 들어왔을거니까)

그럼 나는 {시간 : '2021-11-14-15'} 인 도큐먼트가 없으면 넣어주고, 있으면 업데이트를 하고 싶다.

없으면 넣어주고 있으면 업데이트하는 편리한 연산을 우리는 방금 배웠다. 이런 경우에 `UPSERT - REPLACE`를 쓰면 된다. 1번과 마찬가지로 시간값을 `_key`로 만들어주는게 쿼리 튜닝에 좋다.

왜 `UPSERT - UPDATE`가 아니고 `REPLACE`를 썼을까? 단순히 통계값이 방문자수 하나이면 `UPDATE`가 간편하지만, 통계값이 무수히 많다면 `REPLACE` 하나로 전체 도큐먼트를 통째로 교체하는게 낫다.

**아니면,** [앞서 배운 INSERT의 OPTIONS](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)가 기억나는가? 얘를 써도 좋다.

두 연산이 완전히 동일한 것인지에 대한 의문이 드는데, 이 부분은 Arango Community에 질문 후 답변이 오면 여기에 수정해두도록 하겠다.


## 3. 어디까지 왔나
이제 기초적인 CRUD에 관한 AQL은 다 다룬 것 같다. 나중에 기회가되면 부록처럼 각 연산의 원리와 성능에 대해 추가적으로 적어보려고 한다.

다음 시간부터는 AQL의 꽃인 그래프 관련 연산을 살펴본다. 그래프 횡단부터 시작하여, 각 횡단 단계에서 실행할 수 있는 여러가지 연산들을 배워보자.

아~주 재미있는 시간이 될 것이다 :)

  
1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. [AQL(Arango Query Lang) 배워보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)
5. [AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)
6. **(지금 보고있는 글) AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE**

