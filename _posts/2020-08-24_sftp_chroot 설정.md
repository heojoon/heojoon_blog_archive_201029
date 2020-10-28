# sftp + chroot 설정

---


- **환경** 

  OS :  CentOS Linux release 7.6.1810 (Core)

  ssh : OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017



* **유저 생성**

~~~bash
useradd bob
passwd bob 
~~~



* **유저설정 변경**

~~~bash
[root@localhost home]# useradd bob
[root@localhost home]# passwd bob
[root@localhost home]# vipw
​~~~ 수정
bob:x:1002:1002::/home/bob:/bin/false
​~~~ 
~~~



* **SSH 설정 변경**

~~~bash
[root@localhost home]# vi /etc/ssh/sshd_config
​~~~ 부분 수정
Subsystem       sftp    internal-sftp -f local6 -l info

# Example of overriding settings on a per-user basis 
Match Group bob
        ChrootDirectory /home/bob
        ForceCommand internal-sftp -f local6 -l info
        AllowTcpForwarding no
​~~~ 부분 수정
~~~



* **홈디렉토리 퍼미션 변경 및 업로드 디렉토리 생성**


~~~bash
[root@localhost home]# cd /home
[root@localhost home]# ll
drw------   3 bob   bob     106  8월 23 19:16 bob
[root@localhost home]# chown root bob
[root@localhost home]# chmod 750 bob
drwxr-x---   3 root   bob     106  8월 23 19:16 bob
[root@localhost home]# cd bob
[root@localhost bob]# mkdir upload
[root@localhost bob]# chown bob:bob upload
[root@localhost bob]# ls -ald upload
drwxr-xr-x 2 bob bob 15  8월 23 19:17 upload

~~~



* **로그파일 설정**

~~~bash
[root@localhost bob]# vi /etc/rsyslog.conf
​~~~추가
# sftp log
local6.*                                                /var/log/sftp.log
​~~~
~~~



**※  참고**

> Match Group bob 부분에서 bob 그룹에 매칭될 경우 아래 chroot 정책이 수행되는 것으로 특정 유저로 하거나 관리할 Group을 만들어 준 후  ChrootDirectory /home/%u  로 설정해 주는 것이 좋음

