---
layout: post
title: Jenkins Install guide
tag: jenkins install
subtitle: Jenkins Install guide
---



# Jenkins Install guide



1. 다운로드

    ```bash
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
    ```
    
    ※ 만약 위의 명령어가 실패한다면 재시도를 시간 간격을 두고 한다. 
    
2. 설치

    ~~~bash
    yum -y install jenkins
    ~~~
    
    
    
3. 구동

    ~~~bash
    systemctl enable jenkins
    systemctl start jenkins
    ~~~

    

4. 구성

    * 브라우저로 http://192.168.193.130:8080 접속
    * init 패스워드는 서버의 설치 위치 확인 (브라우저에 경로 정보) (require root)
    * recommand 플러그인으로 구성 진행
    * 완료

[출처] [jenkins 공식 홈페이지](https://jenkins.io/doc/book/installing/)