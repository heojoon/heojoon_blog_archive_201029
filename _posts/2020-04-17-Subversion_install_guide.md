---
layout: post
title: Subversion Install Guide
tag: SVN subversion install
subtitle: Subversion Install Guide
---

# Subversion Install Guide



## Server 구성

#### 1. svnserve single



1. **source download **[[home](http://subversion.apache.org/download.cgi)]
   
   ~~~bash
   $ wget http://mirror.apache-kr.org/subversion/subversion-1.13.0.tar.gz
   $ tar zxvf subversion-1.13.0.tar.gz
   ~~~
   
2. **Requirements**

   * sasl

     ~~~bash
     $ sudo yum install cyrus-sasl cyrus-sasl-devel cyrus-sasl-md5 
     ~~~

   * sqlite

     ~~~bash
     $ cd subversion-1.13.0  # svn 압추 해제 폴더로 이동
     $ wget https://www.sqlite.org/2015/sqlite-amalgamation-3081101.zip
     $ unzip sqlite-amalgamation-3081101.zip 
     $ mv sqlite-amalgamation-3081101 sqlite-amalgamation
     ~~~

   * apr [[home](https://apr.apache.org/download.cgi)]

     ~~~bash
     $ wget http://apache.mirror.cdnetworks.com//apr/apr-1.7.0.tar.gz
     $ tar zxvf apr-1.7.0.tar.gz
     $ cd apr-1.7.0
     $ ./buildconf
     $ ./configure
     $ make
     $ sudo make install
     ~~~

     ~~~
     설치 완료 경로 : /usr/local/apr
     ~~~

   * apr-util [[home](https://apr.apache.org/download.cgi)]

     ~~~bash
     $ sudo yum -y install expat-devel
      # install
      $ wget http://apache.mirror.cdnetworks.com//apr/apr-util-1.6.1.tar.gz
      $ tar zxvf apr-util-1.6.1.tar.gz
      $ cd apr-util-1.6.1
      $ ./configure --with-apr="/usr/local/apr"
      $ make
      $ sudo make install  
     ~~~
     ~~~
      설치완료 경로 : /usr/local/apr/lib , /usr/local/apr/bin/apu-1-config
     ~~~

3. **compile**

   ~~~bash
   $ ./configure --prefix=/app/svn \
   --disable-static \
   --with-lz4=internal \
   --with-utf8proc=internal \
   --with-apr=/usr/local/apr/bin/apr-1-config \
   --with-apr-util=/usr/local/apr/bin/apu-1-config \
   --with-sasl
   $ make
   $ make install
   ~~~
   
4. **make directory**

   ~~~bash
   $ mkdir /app/svn/log /app/svn/run /app/svn/repo /app/svn/conf
   ~~~

5. **make script for startup**

   ~~~bash
   $ vi /app/svn/bin/startup.sh
   
   #!/bin/bash
   
   EXUSER=vmuser
   HOME_DIR=/app/svn
   EnvironmentFile=${HOME_DIR}/conf/svnserve
   LOGFILE="${HOME_DIR}/log/svn.log"
   
   if [ ${UID} != $(id -u ${EXUSER}) ];then
           echo "[Error] Please run as ${EXUSER}"
   else
           source ${EnvironmentFile}
           ${HOME_DIR}/bin/svnserve --log-file=${LOGFILE} --daemon --pid-file=/app/svn/run/svnserve.pid ${OPTIONS} && echo "svnserve started!!" || echo "[Error] svnserve failed start!!"
   fi
   
   $ chmod 755 startup.sh
   ~~~

6. **Make script for shutdown**

   ~~~bash
   $ vi /app/svn/bin/shutdown.sh
   
   #!/bin/bash
   
   EXUSER=vmuser
   HOME_DIR=/app/svn
   EnvironmentFile=${HOME_DIR}/conf/svnserve
   
   if [ ${UID} != $(id -u ${EXUSER}) ];then
           echo "[Error] Please run as ${EXUSER}"
   else
           source ${EnvironmentFile}
           [ ! -e ${HOME_DIR}/run/svnserve.pid ] && { echo "[Error] no pid file,check already stoped" ; exit 1; }
           pkill -F ${HOME_DIR}/run/svnserve.pid && { rm ${HOME_DIR}/run/svnserve.pid ; echo "svnserve stoped!!"; } || echo "[Error] Can't shutdown svn process"
   fi
   
   $ chmod 755 shutdown.sh
   ~~~

7. **Make svn engine config option**
   file : /app/svn/conf/svnserve

   ~~~bash
   $ vi /app/svn/conf/svnserve
   # OPTIONS is used to pass command-line arguments to svnserve.
   #
   # Specify the repository location in -r parameter:
   OPTIONS="--threads -r /app/svn/repo"
   ~~~

8. **SVN startup**

   ~~~
   ./startup.sh
   ~~~

9. **make repository**

   ~~~bash
   $ /app/svn/bin/svnadmin create --fs-type fsfs /app/svn/repo/testprj
   ~~~

10. **Set the repository made config**
      file : /app/svn/repo/testprj/conf/svnserve.conf

   ~~~bash
   $ vi /app/svn/repo/testprj/conf/svnserve.conf
   [general]
   anon-access = none
   auth-access = write
   realm = testprj
   
   [sasl]
   use-sasl = true
   min-encryption = 128
   max-encryption = 256
   ~~~

11. **make user**

    ~~~bash
    $ sudo saslpasswd2 -c -f /app/svn/run/sasldb2 -u testprj admin
    ~~~

12. **show user list**

    ~~~bash
    $ sudo sasldblistusers2 -f /app/svn/run/sasldb2
    ~~~

13. **make svn.conf for sasl2 auth** (require root)
    file : /etc/sasl2/svn.conf

    ~~~bash
    $ vi /etc/sasl2/svn.conf
    pwcheck_method: auxprop
    auxprop_plugin: sasldb
    sasldb_path: /app/svn/run/sasldb2
    mech_list: DIGEST-MD5
    ~~~

14. **change Permission** (require root)
    svnserve 프로세스가 일반유저(vmuser)권한으로 구동되므로 sasldb2의 읽기 권한이 필요하다. 

    ~~~bash
    $ cd /app/svn/run
    $ chown root:vmuser sasldb2
    $ ls 
    -rw-r-----  1 root   vmuser 12288  4월 16 19:38 sasldb2
    -rw-rw-r--  1 vmuser vmuser     5  4월 16 21:29 svnserve.pid
    ~~~

15. **startup SVN**
    svn 서버를 구동한다.

    ~~~bash
    $ cd /app/svn/bin
    $ ./startup.sh
    ~~~

16. **make default directory** 
    svn 프로젝트 수행시 기본적으로 사용하느 디렉토리 생성

    ~~~bash
    $ svn mkdir svn://127.0.0.1/testprj/trunk -m "make directory"
    $ svn mkdir svn://127.0.0.1/testprj/branches -m "make directory"
    $ svn mkdir svn://127.0.0.1/testprj/tags -m "make directory"
    ~~~



### 2. apache + svnserve

만약 클라이언트에서 http , svn 을 통해 SVN을 사용하기 위해서는 apache 설치가 필수다.

1. apache설치

2. svn 설치 

   - 소스 컴파일

     ~~~bash
     $ ./configure --prefix=/app/svn \
     --with-apxs=/app/web/apache/bin/apxs \
     --disable-static \
     --with-lz4=internal \
     --with-utf8proc=internal \
     --with-apr=/usr/local/apr/bin/apr-1-config \
     --with-apr-util=/usr/local/apr/bin/apu-1-config \
     --with-sasl
     $ make
     $ make install
     ~~~

   - 이후 과정은 svnserve single로 설치하는 것과 동일

3. **apache + svn 연동**

   * 인터넷에서 찾아서 보세요. (향후 업데이트 예정)







## Client 구성 ( for Windows & 이클립스)

1. **Install windows subversion binary for SASL**
   [Windows 에서 SASL 인증을 사용 할 수 있도록 Subversion 바이너리 프로그램 설치](http://subclipse.tigris.org/wiki/JavaHL#head-21ee92263236d28f7c0f5fcfd412d90f4d36ac7c)
   https://www.collab.net/downloads/subversion 접속 후 바이너리 버전 다운로드 후 설치 

2. **윈도우 환경설정 변수에서 PATH 설정** 

3. **이클립스 마켓플레이스에서 Svn 설치**

   * help > eclipse marketplace > search > svn 검색 > subclipse 설치 

4. 윈도우 명령행에서 테스트

   ~~~powershell
   c:\> svn info svn://128.11.1.158/testprj
   ~~~

   

















-----

##### ※ 용어정리

*SASL*(Simple Authentication and Security Layer)은 인터넷 프로토콜에서 인증과 데이터보안을 위한 프레임워크이다. 어플리케이션과 프로토콜들로 부터 인증 메커니즘을 분리시킨다.  ( [RFC 스펙 문서](https://tools.ietf.org/html/rfc4422) )

![img](https://k.kakaocdn.net/dn/eOuhNa/btqxyMvRxaV/PXho3SVaXnfx1rCs5zSKt0/img.png)

SASL 을 통해서 인증에 취약한 어플리케이션에 대한 인증을 보증할 수 있는 기술이다.

