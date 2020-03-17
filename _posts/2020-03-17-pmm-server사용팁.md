---
layout: post
title: Mysql모니터링툴 PMM-SERVER
tag: mysql
subtitle: PMM-SERVER 구축시 노하우
---

MySQL 모니터링을 위한 오픈소스를 찾아봤는데 쓸만한게 많이 없더라.. 그중에 PMM-SERVER 라는 모니터링 툴을 찾아서 설치하였다.

PMM-SERVER는 기본적으로 프로메테우스와 그라파나를 베이스로 삼는다. 그래서 그라파나 플러그인 설치를 통해서도 구성할 수 있다.


1. 삭제 
~~~
pmm-admin --server-insecure-tls --server-url=http://${ID}:${PWD}@${SERVER-URL} remove mysql ${SERVICE-NAME}
~~~

위와 같이 삭제 명령어를 날려도 DB에서 노드 정보가 삭제되지 않는다. 
노드 정보까지 삭제하려면 postgresql (내부 인베디드 서버)에 접속해서 삭제해야한다.
