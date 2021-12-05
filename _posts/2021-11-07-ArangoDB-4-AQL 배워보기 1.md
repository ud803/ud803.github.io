---
layout: post
title: <아랑고DB> 4. AQL(Arango Query Lang) 배워보기 1 - WebUI / INSERT
categories: 아랑고DB
tags: [아랑고DB, 그래프DB]
---
  
<div class="message">
이번 글에서는 아랑고DB의 아주 간편한 웹 인터페이스로 시작한다. 웹 UI를 간단히 보고, 아랑고DB의 데이터 타입과 데이터를 삽입하는 방법, 그리고 다양한 설정값에 대해 알아본다.  
</div>

## 1. 들어가기에 앞서
여기서는 아랑고DB에서 제공하는 교육 자료를 사용할 예정이기 때문에 영어를 읽는데 무리가 없다면 해당 자료만 보고 이 글은 패스해도 된다.
자료는 아랑고DB의 공식 튜토리얼 자료 중 하나를 발췌했고, [여기](/public/ArangoDB-GraphCourse_Beginners.pdf)를 눌러 다운받을 수 있다. 더 많은 자료는 [여기](https://www.arangodb.com/learn/)에서 확인하자.

## 2. Web UI 사용해보기
AQL을 사용하고 데이터를 쉽게 조회할 수 있는 가장 간단한 방법은 아랑고DB를 설치하면 자동으로 설치되는 Web UI에 접속하는 것이다. 아무 브라우저나 열어서 아랑고DB가 설치된 서버 주소의 8529 포트로 접속할 수 있으며, 잘 안 되면 [세팅에 대해 적었던 글](https://ud803.github.io/arangodb/2021/11/02/ArangoDB-2/)에서 Web UI 관련 부분을 읽어보자.

나는 AWS EC2를 사용중이기 때문에, `http://ec2-xxx-xxx-xxx-xxx.ap-northeast-2.compute.amazonaws.com:8529` 주소로 접속했다. 
성공적으로 접속했다면 귀여운 아보카도 그림과 함께 로그인 화면이 뜬다. 따로 사용자를 생성해주지 않았다면 계정은 `root`, 비밀번호는 초기 설정한 비밀번호를 입력하자.

입력하면 데이터베이스를 선택하는 드롭다운이 생기는데, 일단은 그대로 `_system` 데이터베이스로 접속한다. 관리자 데이터베이스이기 때문에 서버 상태에 대한 대시보드와 함께 데이터베이스 목록(Databases), 레플리케이션, 서버 Log, 유저 권한을 설정할 수 있는 메뉴 등이 나타난다. 

당장 여기서는 볼 게 없기 때문에, 우측 상단의 `DB:_SYSTEM` 옆의 새로고침 버튼 같은 걸 눌러서 데이터베이스 선택 화면으로 다시 나간 뒤, 저번에 만들었던 `movies` 데이터베이스에 접속해보자. 아까와는 메뉴 구성이 다른 것을 볼 수 있다. 

Collections 메뉴로 들어가면 지난번에 만들었던 `titles` 컬렉션이 있는데, 눌러보면 도큐먼트들의 목록이 쭉 나오는 것을 알 수 있다. 탑바에 있는 `Indexes`, `Info`, `Settings` 들은 컬렉션 수준의 설정을 의미한다. 나중에 인덱스가 정상적으로 만들어졌는지 확인하기 위해 종종 사용한다.

그리고 대망의 Queries 메뉴로 들어가보자. 여기는 AQL을 입력하고 바로 실행시킬 수 있는 환경을 제공해준다. 비록 에디터의 기능이 거의 없다시피해서 수정이 편리하지는 않지만, 쿼리의 실행 관련 Execution Plan과 실행 결과를 곧바로 볼 수 있어 편리하다.

저번 글에서 잠깐 맛만 봤던 <abbr title="Arango Query Language">AQL</abbr>을 테스트해보자. `titles` 컬렉션의 모든 도큐먼트를 반환하는 쿼리다.
  
{% highlight sql %}
FOR title IN titles
    RETURN title
{% endhighlight %}

## 3. AQL(Arangu Query Language) 파헤쳐보기
이제 본격적으로 AQL에 대해 배워보려고 한다. AQL의 [문서](https://www.arangodb.com/docs/stable/aql/)에 따르면, AQL은 선언형 언어로써 어떤 결과를 조회 할지를 표현하는 언어이다. 목적성에 있어서 SQL과 유사하지만, 큰 차이점이 있다면 **AQL은 DML(Data Manipulation Lang)이지, DDL(Data Definition Lang)이나 DCL(Data Control Lang)이 아니**라는 점이다. 즉, 데이터를 읽고 수정은 하지만, 데이터베이스를 생성하거나 인덱스를 생성하거나 하는 연산은 지원하지 않는다. (아랑고DB를 의미하는 게 아닌, AQL에서 지원하지 않는다는 것!) 즉 SQL에서는 `CREATE DATABASE ...`, `CREATE TABLE ...`과 같은 DDL 구문이 있지만 AQL에는 없다는 것임!

기본 연산을 배우기 전에, 아랑고DB의 [데이터 타입](https://www.arangodb.com/docs/stable/aql/fundamentals-data-types.html)을 잠깐 짚고 넘어가자.

### 데이터 타입
아랑고DB는 데이터 타입을 `primitive`와 `compound`로 분류한다. 전자는 하나의 값을 가지는 데이터 형태이고, `null`, `boolean`, `number`, `string` 형태가 있다. 후자는 0개~ 여러 값을 포함하는 데이터 타입으로, `array/list`와 `object/document`가 있다.
  
오브젝트는 중괄호 `{}`로 감싸진 데이터 타입이고, 이름-값 쌍으로 구성되어 있으며, 각 쌍은 컴마`,`로 분리된다.

즉, 아래의 형태이다. 이름에 해당하는 name, age, blog는 따옴표나 쌍따옴표로 감싸도 되고, 감싸지 않아도 된다. 
{% highlight js %}
{
  'name' : 'Scott',
  'age' : 29,
  'blog' : 'ud803.github.io' 
}
{% endhighlight %}
  
글에서 계속해서 나오는 도큐먼트는 컬렉션에서 최상위 데이터 레벨을 의미한다. 즉, 오브젝트나 도큐먼트나 같은 형태이지만 컬렉션에 들어가는 레코드로써의 의미를 위해 도큐먼트라는 표현을 쓴다. 하나의 도큐먼트 내에 또다른 도큐먼트 형태가 있다면 (즉, nested object) 보통 오브젝트라고 분리하여 부르는 것 같다. 
  
### CREATE
<div class="exclamation">
아래 예시를 따라하기 전에, Collections에서 titles를 선택하고, 탑바의 Settings를 눌러 데이터를 Truncate 해주자.
그리고 탑바의 Indexes에서 +를 누르고, Persistent Type을 선택, Fields와 Name에는 title을 입력하고, Unique를 체크하고 Create을 눌러주자.
이는 컬렉션에서 title 필드에 인덱스를 추가하고, 값이 고유하도록 constraint을 주는 작업이다!
</div>

<div class="warning">
컬렉션 설정에서 Delete는 컬렉션을 아예 통째로 없애는 것이고, Truncate은 나머지 설정값은 그대로 둔 채 도큐먼트들만 지운다.
</div>
  
`INSERT`는 말그대로 특정 컬렉션에 도큐먼트를 삽입하는 연산이며, 아래와 같이 쓰인다.
  
{% highlight sql %}
// 기본 형태
INSERT _document_ INTO _collection_
{% endhighlight %}
{% highlight sql %}
// 아래 명령은 한 줄씩 입력해야한다
INSERT { title: "Avengers 1" } INTO titles
INSERT { "title" : "Avengers 2" } INTO titles
{% endhighlight %}
위 AQL에서 왜 명령을 한 줄씩 입력해야할까? AQL은 **컬렉션 하나당 한 번의 Insert 연산만을 허용하기 때문이다.** [요기](https://www.arangodb.com/docs/stable/aql/operations-insert.html) 맨 첫 부분에 나와있음 
 
{% highlight sql %}
// 1..100이라는 표현은 1부터 100까지 반복하는 For 문법의 한 예시이다
// CONCAT은 타입에 관계없이 두 값을 문자열 형태로 합친다
// 위에서 Avengers 1, 2를 생성했다면 아래 AQL은 실패하는 게 정상이다
FOR i IN 1..100
  INSERT { "title" : CONCAT("Avengers ", i) }  INTO titles
{% endhighlight %}

앞서 `title`에 unique constraint을 주었기 때문에, Avengers 1을 생성하려는 시도는 실패했다. 하지만 때로는 중복인 도큐먼트는 무시하고 알아서 잘 센스있게 나머지 도큐먼트만 넣어주었으면 하는 바람이 든다. 이때 사용할 수 있는 것이 AQL의 Options 개념이다. [여기](https://www.arangodb.com/docs/stable/aql/operations-insert.html#overwritemode)에서 Insert와 관련된 추가 설정들을 잘 보여주고 있는데, 중복을 무시하기 위해서는 `ignoreErrors` 옵션을 사용하면 된다.
  
{% highlight sql %}
FOR i IN 1..100
  INSERT { "title" : CONCAT("Avengers ", i) }  INTO titles
OPTIONS {ignoreErrors: True}
{% endhighlight %}
 
여기서 더 나아가, `overwriteMode` 옵션을 설정하면, 중복 도큐먼트를 마주했을 때의 규칙을 설정할 수도 있다.
- `ignore` : 그냥 무시한다
- `replace` : 덮어쓴다
- `update` : 지정한 도큐먼트의 일부 부분만 업데이트한다
- `conflict` : 에러를 일으킨다. 이는 따로 설정값이 없을 때의 기본 규칙임
  
그 외에 컬렉션 수준의 락을 거는 `exclusive` 옵션과 같은 유용한 설정이 있으니 시간이 되면 살펴보자. 나중에 아랑고DB에 수많은 도큐먼트를 벌크로 넣을 때, 이런 세세한 옵션들의 존재 유무를 알아두는 것이 큰 도움이 된다. 

<div class="tip">
물론 시작부터 문서를 보면서 모든 설정을 달달달 외우는 것은 아무 필요가 없다. 어떤 설정이 있는지만 슥 훑어보고 정말 필요할 때 가져다 쓰면 되는 것임
</div>
 
## 4. 어디까지 왔나
생각보다 쓸 내용이 많아져서 AQL 기초 관련 내용을 1편과 2편으로 나누려고 한다. 2편은 도큐먼트를 삽입하는 또다른 방법인 UPSERT를 알아보고, 나머지 Read, Update, Delete 를 간단히 짚고 넘어가려고 한다. 그리고 그래프 횡단은 내용이 방대하기에 따로 분리할 예정이다.

1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. **(지금 보고있는 글) AQL(Arango Query Lang) 배워보기 1**
5. [AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)
6. [AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/14/ArangoDB-6-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-3/)
7. [그래프 개념잡기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/23/ArangoDB-7-%EA%B7%B8%EB%9E%98%ED%94%84-%EA%B0%9C%EB%85%90-%EC%9E%A1%EA%B8%B0/)
8. [그래프 횡단하기 Graph Traversal](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/12/05/ArangoDB-8-%EA%B7%B8%EB%9E%98%ED%94%84-%ED%9A%A1%EB%8B%A8%ED%95%98%EA%B8%B0-Graph-Traversal/)
