---
layout: post
title: <아랑고DB> 8. 그래프 횡단하기 Graph Traversal
categories: 아랑고DB
tags: [아랑고DB, 그래프DB]
---
  
<div class="message">
이번 글에서는 드디어 그래프 횡단에 대해 배워보려고 한다. 사실상 그래프DB를 사용하는 가장 큰 이유가 바로 이 그래프 횡단의 유용함이라고 생각한다.

재밌게 배워서 알차게 써먹어보자!
</div>

## 1. 준비사항

마찬가지로 아랑고DB에서 제공해주는 공식 튜토리얼 문서를 기반으로 설명을 할 예정이다. 튜토리얼에서 사용되는 비행 관련 데이터셋은 [여기](/public/ArangoDB-GraphCourse_Beginners.pdf)를 눌러 다운받을 수 있다.

이 데이터는 각 공항들 사이에 존재하는 비행 경로를 그래프로 나타낸 데이터이다.

PDF파일의 24페이지를 보면, Airports 데이터셋을 import하는 방법이 나와있다. ArangoDB가 설치되어 있는 Ubuntu Shell에서 내가 진행한 방식은 아래와 같다.

curl 명령어는 [이 사이트](https://linuxize.com/post/curl-command-examples/)에 간결하게 잘 나와있어 참고했다.

{% highlight shell %}
# 임의의 경로에서 curl로 다운받는다
# -L 옵션은 리다이렉션을 끝까지 따라가는 옵션이고, 여기서의 결과를 --output에 저장한다
# 원래 일반적인 url은 curl -O {url} 하면 되는데, 얘는 리다이렉션이 걸려있어서 이렇게 다운받음
sudo curl -L https://www.arangodb.com/arangodb_graphcourse_demodata/ --output airport.zip

sudo unzip airport.zip

# airports 노드 컬렉션 임포트
arangoimport --file airports.csv --collection airports --create-collection true --type csv

# flights 엣지 컬렉션 임포트
# 아래 명령어는 컴퓨터 성능에 따라 꽤 오랜 시간이 걸릴 수 있습니다!
# ec2 t2.micro에서는 한참 걸리네요
arangoimport --file flights.csv --collection flights --create-collection true --type csv --create-collection-type edge
{% endhighlight %}

대량의 데이터가 있다면, 이렇게 csv 형태를 편하게 import 할 수 있다는 것도 기억해두자.


## 2. 데이터 살펴보기

데이터 임포트가 완료되고 나면, 아랑고 WebUI로 가서 데이터를 살펴보자. `_system` 데이터베이스에 디폴트로 생성되었고, (위 명령어에서 데이터베이스를 지정해주면 새로운 데이터베이스에 만들 수도 있다.) 각각 `airports`와 `flights` 컬렉션에 생성되었다.

`airports`는 5개 국가의 공항에 대한 위치 정보를 가지고 있고, 3375개의 노드 도큐먼트로 구성되어 있다.

`flights`는 위 공항들을 잇는 엣지 도큐먼트이고, 출발 시간, 도착 시간, 항공편 번호 등의 비행에 관한 데이터를 가지고 있다. 약 44만 개의 데이터가 존재한다.

이제 이 데이터들을 기반으로 그래프 횡단에 대해 배워보자.

## 3. 그래프 횡단 Grpah Traversal

그래프 횡단이란, 그래프의 엣지를 따라 움직이는 행위를 지칭한다. 이때 횡단에서 몇 개의 엣지를 이동하는지를 **횡단의 깊이 Traversal Depth**라고 부른다.

아랑고 튜토리얼에서 발췌한 아래의 그림을 보면 이해하기 쉽다.

![Node and Edges](/public/img/arango-depth.png){: width="100%" height="100%"}

모든 그래프 관련 데이터베이스에서 사용하는 횡단 관련 문법은 상이하다. 아랑고DB의 AQL에서는 아래와 같은 문법을 사용한다.

{% highlight sql %}
FOR vertex[, edge[, path]]
  IN [min[..max]]
  OUTBOUND|INBOUND|ANY startVertex
  edgeCollectionName[, more...]
{% endhighlight %}

위 문법을 그대로 해석하면, 아래 정도가 되겠다.

> startVertex를 출발점으로 잡고, 이 출발점과 연결되어 있는 edgeCollectionName에 연결된 엣지 중에서, 출발점에서 뻗어나가거나(OUTBOUND) | 들어오거나(INBOUND) | 둘 중 하나거나(ANY) 에 해당하는데, 깊이가 min~max 사이인 경로에 해당하는 값들을 리턴해라. 이때, 리턴하는 값들은 도착하는 노드, 엣지, 경로이다. 

천천히 하나씩 뜯어보자. 일단 위 쿼리에서 대괄호[]안의 내용은 생략이 가능하다는 의미이며, 세로 라인 |은 또는(or)의 의미로써 상황에 맞는 것을 사용하면 된다는 뜻임

### FOR vertex[, edge[, path]]

횡단에서 사용할 세 개의 변수 vertex, edge, path를 나타내는 값이다. `FOR loop`에서 썼던 것처럼 변수 이름은 사용자 마음임. 

이때 `vertex`는 **횡단 후에 도착해 있는** 노드 오브젝트를 의미하며, `edge`는 횡단하게 될 엣지, `path`는 횡단하게 될 경로의 모든 노드, 엣지를 총칭한다.

여기서 `edge`와 `path`는 사용해도 되고, 사용하지 않아도 된다. 

{% highlight sql %}
// JFK 공항에서 출발했을 때, 도착하는 공항들을 리턴함
LET jfk = Document('airports/JFK')
FOR v IN OUTBOUND jfk flights
  RETURN v
{% endhighlight %}

{% highlight sql %}
// JFK 공항에서 출발했을 때, 도착하는 공항들과 그 비행 정보를 리턴함
LET jfk = Document('airports/JFK')
FOR v, e IN OUTBOUND jfk flights
  RETURN {v, e}
{% endhighlight %}

### IN [min[..max]] OUTBOUND|INBOUND|ANY startVertex

여기서 `startVertex`는 횡단의 출발점을 의미하며, 이 출발점에서 나가는 방향을 OUTBOUND, 들어오는 방향을 INBOUND, 둘 다 상관없이 연결되어 있기만 하면 ANY라고 지칭한다. 

셋 중 하나를 골라서 쓰면 된다.

그리고 `min`, `max`는 생략 가능한데, 횡단의 깊이를 나타낸다. 생략하면 디폴트로 1..1로 설정되어 있음. 

### edgeCollectionName[, more...]

마지막으로 `edgeCollectionName`들은 횡단의 기준이 되는 엣지 컬렉션의 이름을 의미한다. 하나만 써도 되고, 컴마로 분리하여 여러 엣지를 쓸 수도 있다.



## 4. 어디까지 왔나

그래프 횡단에 들어가기 위한 모든 준비를 끝마쳤다. 다음 글에서는 진짜진짜 재미있는 그래프 횡단을 진짜진짜 들어가려고 한다.

기존 데이터베이스들에서는 감히 상상도 할 수 없었던 것들을 많이 실습해보자!

  
1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. [아랑고DB 세팅하기 on Ubuntu](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/02/ArangoDB-2-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-on-Ubuntu/)
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. [AQL(Arango Query Lang) 배워보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)
5. [AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)
6. [AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/14/ArangoDB-6-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-3/)
7. **(지금 보고있는 글) 그래프 개념잡기**
