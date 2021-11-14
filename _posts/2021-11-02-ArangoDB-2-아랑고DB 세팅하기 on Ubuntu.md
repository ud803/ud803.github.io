---
layout: post
title: <ArangoDB> 2. 아랑고DB 세팅하기 on Ubuntu 
categories: 아랑고DB
tags: [아랑고DB, 그래프DB]
---
  
<div class="message">
이번 글에서는 아랑고DB를 세팅하는 방법에 대해 알아보려고 한다. 로컬(본인 컴퓨터나 노트북)에서 작업해도 좋고 개발용 서버가 있다면 서버에서 해도 좋다. 나는 AWS에서 제공하는 프리티어 EC2를 사용 중이다.
</div>

## 1. 설치하기
  설치는 매우 간단하다. [아랑고DB 공식 홈페이지](https://www.arangodb.com/download-major/)에 있는 커뮤니티 버전 중, 본인 환경에 맞는 것을 설치하면 된다. 여기서는 Ubuntu 버전을 설치한다.
  
{% highlight bash %}
// 아래 curl 명령어가 오류난다면 권한 문제일 경우가 많다. sudo를 붙이거나 폴더의 권한을 바꿔주자.
curl -OL https://download.arangodb.com/arangodb38/DEBIAN/Release.key
sudo apt-key add - < Release.key
                                
echo 'deb https://download.arangodb.com/arangodb38/DEBIAN/ /' | sudo tee /etc/apt/sources.list.d/arangodb.list
sudo apt-get install apt-transport-https
sudo apt-get update
                                
// 약 300mb 정도의 여유 공간이 필요하다
// 설치 화면이 나타나면, root 권한의 비밀번호를 설정하게 된다. 꼭 기억해두자.
// 자동 업데이트나 업데이트 시 백업 여부는 개발용이기 때문에 No,No로 해두었다.
sudo apt-get install arangodb3=3.8.2-1
{% endhighlight %}

이제 설치가 완료되었다. `arangosh` 명령어를 통해 라이브된 아랑고DB 쉘에 접속할 수 있고, 혹은 `service arangodb3 status` 명령어를 통해 서비스에 등록되어 정상 설치된 것을 확인할 수 있다.
                                
## 2. 설치 경로에 관한 이야기

우분투에서 프로그램이 설치되는 경우, 주로 찾아보는 경로를 잠깐 정리해보자면, 아래와 같다. 이 경로들은 대부분 다른 프로그램들에서도 공통으로 쓰이며, 대개 `/etc` 경로의 해당 config 파일을 열어보면 자세한 설정들이 나와있다.
- `/etc/arangodb3/` : 각종 설정 파일이 저장된 공간
- `/var/lib/arangodb3` : 아랑고DB의 파일들이 저장되는 곳
- `/var/log/arangodb3` : 아랑고DB의 로그가 쌓이는 곳

## 3. 설정 파일과 Web UI 세팅하기
이제 아랑고DB의 설정 파일을 한 번 살펴보자.
                                
{% highlight bash %}
cd /etc/arangodb3
sudo vim arangod.conf
{% endhighlight %}   
                                
여기서 server 항목을 살펴보면, endpoint가 tcp:127.0.0.1:8529로 설정된 것을 볼 수 있다.
실제로 아랑고DB의 Web UI에 접속하려면 브라우저에 본인의 서버 주소에 8529 포트를 붙여 접속할 수 있다. 예를 들어, EC2를 쓰는 내 경우 `ec2-x-x-x-x.ap-northeast-2.compute.amazonaws.com:8529`로 붙을 수 있다.

하지만 이때 접속이 되지 않는다면, 
1) endpoint를 0.0.0.0:8529로 변경해본다. 이에 관한 자세한 내용은 [여기](https://superuser.com/questions/949428/whats-the-difference-between-127-0-0-1-and-0-0-0-0)를 읽어보기
2) 그래도 안 된다면 본인 서버의 보안을 확인한다. (EC2의 경우 보안그룹의 inbound traffic이 내 주소에 열려있는지..)
3) 그래도 안 된다면 [아랑고DB의 가이드](https://www.arangodb.com/docs/stable/troubleshooting-arangod.html)를 살펴본다.
                                
위 과정을 거쳐서 다시 브라우저로 접속해보면 아랑고 Web UI가 설치된 것을 확인할 수 있다! 아랑고DB의 강력하고 편리한 기능이기 때문에 얼굴을 기억해두자.

**초기 세팅했던 root 계정과 비밀번호로 접속하면** `_system` 데이터베이스로 접속할 수 있다. `_system` db는 다른 데이터베이스를 생성하고, 시스템 관련 설정을 할 수 있는 기본 데이터베이스라고 보면 된다.
                     
## 4. 어디까지 왔나
다음 글에서는 아랑고DB의 데이터 구조에 대해 간단하게 알아보고, 아랑고 쉘에서 명령어를 통해 여러가지 CRUD 작업을 해서 실체를 익혀보려고 한다.
  
1. [아랑고DB란? 왜 쓰는가?](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/10/31/ArangoDB-1-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0/)
2. **(지금 보고있는 글)아랑고DB 세팅하기 on Ubuntu**
3. [아랑고DB 쉘로 붙어서 명령어 체험해보기, 실체 파악해보기](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/06/ArangoDB-3-%EC%95%84%EB%9E%91%EA%B3%A0DB-%EC%89%98-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0/)
4. [AQL(Arango Query Lang) 배워보기 1](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/07/ArangoDB-4-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-1/)
5. [AQL(Arango Query Lang) 배워보기 2 - RETURN / UPDATE](https://ud803.github.io/%EC%95%84%EB%9E%91%EA%B3%A0db/2021/11/10/ArangoDB-5-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-2/)
6. [AQL(Arango Query Lang) 배워보기 3 - REPLACE / UPSERT / REMOVE](https://ud803.github.io/2021/11/14/ArangoDB-6-AQL-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EA%B8%B0-3/)
