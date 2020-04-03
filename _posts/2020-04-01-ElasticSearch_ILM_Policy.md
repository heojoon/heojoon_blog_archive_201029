---
layout: post
title: Elasticsearch_Index_LifeCyle_관리
tag: ELK elasticsearch
subtitle: Elasticsearch Index LifeCyle 관리
---

# Elasticsearch Index LifeCyle 관리

Elasticseach 에서 오래된 인덱스를 삭제하기 위한 방법을 찾아봤다. logstash 포맷형식으로 fluent-bit을 통해서 elasticseach 로 데이터를 적재하고 있는 상황이다.



### 1.  Elasticseach 6.6 버전 이전

 - curator라는 cli 툴을 받아서 action_file을 만들어서 관리할 수 있다.

~~~bash
curator
helps manage Elasticsearch time-series indices. It provides an easy way to perform index administration tasks, such as managing aliases, optimizing indices, changing the replica count and modifying index allocation using routing tags.
~~~

* [curator man page](https://manpages.debian.org/testing/elasticsearch-curator/curator_cli.1.en.html)

* [curator official download link](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/yum-repository.html#_signing_key_2)

* [action_file 작성 블로그 1 참고](https://cyuu.tistory.com/157)
* [action_file 작성 블로그 2 참고](https://bkjeon1614.tistory.com/317)



### 2. Elasticsearch 7.6 버전 이후

- ILM(Infomation Lifecyle Management) 기능을 ( X-Pack(유료) 기능) 을 무료 제공 한다.(참고 [elastic 공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html#index-lifecycle-management) )

*  [elaticsearch automate rollover](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html#ilm-gs-apply-policy)
*  설치 버전이 7.6 이기 때문에 이걸로 진행 (EL 7.6에 기존 유료기능이었던 X-pack 중 일부기능 ILM, Login 등 .. 이 무료화 되었다 )
*  아래 내용을 진행 할 경우 kibana의 dev_tool 을 활용하는 것을 권장함

#### 2.1. 라이프 사이클 정책 생성

~~~javascript
PUT _ilm/policy/fluentbit_policy
{
  "policy": {
    "phases": {
      "hot": {                      
        "actions": {
          "rollover": {
            "max_size": "5GB",
            "max_age": "7d"
          }
        }
      },
      "delete": {
        "min_age": "7d",       
        "actions": {
          "delete": {}              
        }
      }
    }
  }
}
~~~



#### 2.2. 라이플 사이클 정책을 적용할 인덱스 템플릿 생성

~~~javascript
PUT /_template/flunetbit_template?pretty
{

  "index_patterns": ["fluentbit-*"],                 
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1,
    "index.lifecycle.name": "fluentbit_policy",      
    "index.lifecycle.rollover_alias": "fluentbit"    
  }
}
~~~



#### 2.3.  최초 기준을 삼아서 정책을 시작할 인덱스 설정

~~~javascript
PUT /fluentbit-2020.04.01?pretty
{
  "aliases": {
    "fluentbit": {
      "is_write_index": true
    }
  }
}
~~~



#### 2.4. 진행상태 확인

~~~javascript
GET /fluentbit-*/_ilm/explain?pretty
~~~



#### 2.5. Kibana Web UI 에서 확인

위의 내용들을 수행하면 정책이 적용 되었다.  Kibana와 연동을 했다면, kibana WEB-UI에서 Management > Elasticsearch > Index Lifecyle Policies 메뉴을 선택하면 위의 과정을 통해서 생성한 내용을 볼수 있다.

<img src="../img/2020-04-01 15_47_29-Kibana.png" alt="2020-04-01 15_47_29-Kibana" style="zoom:50%;" />

그런데 의아한게 Linked indices 가 0 이다. 이는 우리가 Index를 ILM 정책에 적용한 것이 아니라 템플릿을 적용했기 때문에 그렇다. 

<img src="../img/2020-04-01 15_47_50-Kibana.png" alt="2020-04-01 15_47_50-Kibana" style="zoom:50%;" />

적용한 템플릿은 ILM 정책 우측에 Actions 에서 Add Policy to Index template 를 클릭하고 Index template 리스트 박스를 누르면 fluentbit_template 라는 템플릿이 보인다. 이걸 선택하면 Template already has policy 로 이미 적용되어 있다는 경고 문구를 볼 수 있다. 

<img src="../img/2020-04-01 15_48_15-Kibana.png" alt="2020-04-01 15_48_15-Kibana" style="zoom:50%;" />






