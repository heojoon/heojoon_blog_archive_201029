

1. 리눅스 커널/네트워크 매개변수 설정 

 1) /etc/sysctl.conf 파일 수정

vi /etc/sysctl.conf  -> 마지막줄부터 아래 내용 추가


################### 복사 시작 ####################

# icmp redirects를 보내지 않는다.

net.ipv4.conf.eth0.accept_redirects=0

net.ipv4.conf.lo.accept_redirects=0

net.ipv4.conf.default.accept_redirects=0

net.ipv4.conf.all.accept_redirects=0

net.ipv4.conf.eth0.send_redirects = 0

net.ipv4.conf.lo.send_redirects = 0

net.ipv4.conf.default.send_redirects = 0

net.ipv4.conf.all.send_redirects = 0

# proxy arp를 설정하지 않는다.

net.ipv4.conf.eth0.proxy_arp=0

net.ipv4.conf.lo.proxy_arp=0

net.ipv4.conf.default.proxy_arp=0

net.ipv4.conf.all.proxy_arp=0

# 게이트웨이로부터의 redirect를 허용하지 않음으로써 스푸핑을 막기 위해 설정한다.

net.ipv4.conf.eth0.secure_redirects=0

net.ipv4.conf.lo.secure_redirects=0

net.ipv4.conf.default.secure_redirects=0

net.ipv4.conf.all.secure_redirects=0

# 스푸핑을 막기 위해 source route 패킷을 허용하지 않는다.

# 소스 라우팅을 허용할 경우 악의적인 공격자가 IP 소스 라우팅을 사용해서 목적지의

# 경로를 지정할 수도 있고, 원래 위치로 돌아오는 경로도 지정할 수 있다. 이러한 소스 라우팅이

# 가능한 것을 이용해 공격자가 마치 신뢰받는 호스트나 클라이언트인 것처럼 위장할 수 있는 것이다.

net.ipv4.conf.eth0.accept_source_route=0

net.ipv4.conf.lo.accept_source_route=0

net.ipv4.conf.default.accept_source_route=0

net.ipv4.conf.all.accept_source_route=0

# Broadcast로부터 오는 핑을 차단함(Smurt 공격을 차단함).

net.ipv4.icmp_echo_ignore_broadcasts=1

# IP 나 TCP 헤더가 깨진 bad icmp packet을 무시한다.

net.ipv4.icmp_ignore_bogus_error_responses = 1

# 자신의 네트워크가 스푸핑된 공격지의 소스로 쓰이는 것을 차단한다.

# 모든 인터페이스에서 들어오는 패킷에 대해 reply를 하여 들어오는 인터페이스로 나가지 못하는 패킷을 거부한다.

net.ipv4.conf.eth0.rp_filter=2

net.ipv4.conf.lo.rp_filter=2

net.ipv4.conf.default.rp_filter=2

net.ipv4.conf.all.rp_filter=2

# bootp 패킷을 허용하지 않는다.

net.ipv4.conf.eth0.bootp_relay=0

net.ipv4.conf.lo.bootp_relay=0

net.ipv4.conf.default.bootp_relay=0

net.ipv4.conf.all.bootp_relay=0

# 스푸핑된 패킷이나 소스라우팅, Redirect 패킷에 대해 로그파일에 정보를 남긴다.

net.ipv4.conf.eth0.log_martians=1

net.ipv4.conf.lo.log_martians=1

net.ipv4.conf.default.log_martians=1

net.ipv4.conf.all.log_martians=1

# 1/100초에 받아들이는 igmp "memberships"의 수

net.ipv4.igmp_max_memberships=1

# 매우 복잡한 사이트에서는 이 값을 늘리는 것도 가능하지만 64로 두는 것이 적당하며

# 더 늘렸을 경우에는 큰 문제가 발생할 수도 있다.

net.ipv4.ip_default_ttl=64

# 게이트웨이 서버가 아닌 이상 패킷을 포워딩 할 필요는 없다.

net.ipv4.ip_forward=0

# fragmented packet이 메모리에 존재하는 시간을 15초로 설정한다.

net.ipv4.ipfrag_time=15

# SYN_Flooding 공격에 대한 대비로 백로그큐(Backlog Queue)가 가득차면 다른 접속 요구를 받아들이지 못한다.

net.ipv4.tcp_max_syn_backlog = 1024

# TCP 연결에서 Three-way Handshake가 성공적으로 이루어지지 않으면 더 이상 소스 경로를 거슬러 올라가지 않도록한다.

# 따라서 적절한 연결 요청에 대해서만 연결을 맺는다.

# syncookies가 작동할 때 SYN Flooding 공격이 있으면 messages 파일에 아래와 같은 내용이 출력된다.

# possible SYN flooding on port 80. Sending cookies.

net.ipv4.tcp_syncookies = 1

# 일정한 시간과 IP별로 보내고 받는 SYN 재시도 횟수를 3회로 제한한다.

# 이 옵션은 스푸핑된(위조된) 주소로 오는 SYN 연결의 양을 줄여준다.

# 기본 값은 5(180 초에 대응)이며 255를 넘지 않아야 한다.

net.ipv4.tcp_syn_retries = 3

# passive TCP 접속시도가 재접속을 하기 위한 SYNACKs의 값을 정한다. 255 보다 높

# 게 지정할 수 없다. 기본값은 5이며, 180초에 대응이 된다.

net.ipv4.tcp_synack_retries = 3

# 무언가 문제가 있을 때 연결을 위해 재시도 할 횟수, 최소 값과 기본 값은 3이다.

net.ipv4.tcp_retries1=3

# TCP 연결을 끊기 전에 재시도할 횟수.

net.ipv4.tcp_retries2=7

# 연결을 종료시 소요되는 시간을 줄여준다(기본 설정값: 60).

net.ipv4.tcp_fin_timeout=20

# 동시에 유지 가능한 timewait 소켓의 수이다.

# 만약 지정된 숫자를 초과하였을 경우에는 timewait 소켓이 없어지며 경고 메시지가 출력된다.

# 이 제한은 단순한 DoS 공격을 차단하기 위해 존재하는데, 임의로 이 값을 줄여서는 안되며

# 메모리가 충분하다면 적절하게 늘려주는 것이 좋은데, 64M 마다 180000으로 설정하면 된다.

# 따라서 256M일 경우에는 256/4=4 4*180000=720000

# 64M -> 180000

# 128M -> 360000

# 256M -> 720000

# 512M -> 1440000

# 1G -> 2880000

# 2G -> 5760000

#net.ipv4.tcp_max_tw_buckets = 180000

# 연결이 끊어졌다고 판단할 때까지, 얼마나 keepalive probe 를 보낼지 결정. 기본값 9회

# 간단한 DoS 공격을 막아준다.

net.ipv4.tcp_keepalive_probes=2

# keepalive 가 활성되 되어 있을 경우, 얼마나 자주 TCP 가 keepalive 메세지를 보

# 내게 할 것인지를 설정.

net.ipv4.tcp_keepalive_time=30

# keepalive_probes 를 보낼 간격을 정함. probe 를 보낸 후, probes * intvl 의 시

# 간이 지나도록 응답이 없으면 연결이 해제된 것으로 간주하게 됨. 기본 값의 사용

# 시 11분 15초 동안 재시도를 하고 연결을 취소함. 값은 초단위

net.ipv4.tcp_keepalive_intvl = 10

# 서버 쪽에서 닫은 TCP 연결을 끊기 전에 확인하는 횟수를 정한다. 기본 값은 7 로

# RTO 50 초에서 16 분 사이에 해당한다. 웹 서버가 운영 중 이라면 이 값을 줄여서

# 소켓 등이 귀한 리소스를 소비하지 않도록 할 수도 있다.

net.ipv4.tcp_orphan_retries = 2

# SYN 패킷을 전송한 후에 로스가 발생을 하여 ACK 를 일부 받지 못했을 경우, 선택

# 적으로 (selected) 받지못한 ACK 만 받도록 요청하는 것을 허락한다. 로스가 많은

# 네트워크에서는 상당히 중요한 역할을 한다.

net.ipv4.tcp_sack = 1
 
 
 
 
