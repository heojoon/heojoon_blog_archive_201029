---
layout: post
title: FTP Passive Mode 설정 및 사용하기
tag: ftp passivemode
---

(1) FTP 서버 Passive Mode 설정


// 인터넷에 여려 설정이 있지만 아래 설정이 가장 적합했다.
// ※ CentOS7 Chroot Error Fix 확인 (위는 Passive Mode만 가능하도록 한 세팅이다.)



~~~
#
# Custom setting
#

## Default
anonymous_enable=NO
local_enable=YES
write_enable=YES
download_enable=YES
cmds_allowed=ABOR,CWD,LIST,MDTM,MKD,NLST,PASS,PASV,PORT,PWD,QUIT,RETR,RMD,RNFR,RNTO,SITE,SIZE,STOR,TYPE,USER,ACCT,APPE,CDUP,HELP,MODE,NOOP,REIN,STAT,STOU,STRU,SYST
local_umask=022
#dirmessage_enable=YES


## Logging
xferlog_enable=NO
xferlog_file=/var/log/vsftpd.log
xferlog_std_format=NO
dual_log_enable=YES
log_ftp_protocol=YES
session_support=YES



idle_session_timeout=600


#################################################################################
#  Connection 
#################################################################################
#
listen_ipv6=NO
listen=YES
listen_port=10021

## Active Mode
port_enable=NO
#connect_from_port_20=YES
#ftp_data_port=10023  # connect_from_port=YES 일 경우 data전송 port 변경

## PassiveMode
## 데이터 전송을 위해서 Passive mode를 사용할 것인지 설정합니다. (기본값 = YES)
## => Active Mode로 접근할 수 없는 사용자들을 위해 활성화 하는것이 좋습니다.

pasv_enable=YES
pasv_min_port=10015
pasv_max_port=10019

accept_timeout=60
# 패시브 모드(Passive Mode)를 사용하는 클라이언트의 접속 허용시간을 설정합니다. 클라이언트의 요청패킷(SYN Packet)을 받은뒤, 지정된 시간내에 접속이 안될경우 종료합니다.


## 동일한 IP주소에서 이루어지는 데이터 연결을 보장해주는 보안체크 기능을 사용할 것인지
## 설정합니다. (기본값 = NO)
pasv_promiscuous=NO


## Security
chroot_local_user=YES
chroot_list_enable=NO
chroot_list_file=/etc/vsftpd/chroot_list
ls_recurse_enable=NO
pam_service_name=vsftpd
userlist_enable=YES
#tcp_wrappers=YES
#chmod_enable=NO
#download_enable=NO

# CentOS7 Chroot Error Fix
allow_writeable_chroot=YES
~~~




열심히 ftp passive mode 서버 설정을 했다. 이유인즉슨 대상 클라이언트가 Windows CMD 창만 가능하기 때문!

그런데!!!

Windows에서 CMD에서 제공해주는 FTP 프로그램에서 Passive mode 파일전송을 지원하지 않는다!!

윈도우 CMD창에서 ftp 접속을 한후 

~~~
quote pasv
quote list 
~~~

위와 같이 치면 list 명령어(dir)가 수행되어야 하는데 멍때리다가 connection disconnected 가 떨어짐.

passive mode 에서는 명령어 수행하고자 할 경우 quote help 를 쳐서 나오는 명령어들만 사용할 수 있다.

일반적인 명령어는 active 명령어라 권한오류(ftp 서버가 only passive mode 일 경우) 가 나온다.

열심히 구글링을 해봤더니 windows 모든 버전의 ftp client 는 passive mode 지원이 안된다고 한다. 

그래서 사람들이 많이 쓰는 filezila 를 사용해서 스크립트 방식으로 사용하려고 했는데 이것도 안된다!!!! -_-

그래서 ... 이번엔 Winscp를 깔았다.. 이걸로 할거면 ftp를 사용할 이유가 없지.. sftp를 쓰지..
