## 리눅스 커널/네트워크 매개변수 설정

1. icmp redirects 차단.
~~~
net.ipv4.conf.eth0.accept_redirects=0
net.ipv4.conf.lo.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.eth0.send_redirects = 0
net.ipv4.conf.lo.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.send_redirects = 0
~~~

2. proxy arp를 미설정.
~~~
net.ipv4.conf.eth0.proxy_arp=0
net.ipv4.conf.lo.proxy_arp=0
net.ipv4.conf.default.proxy_arp=0
net.ipv4.conf.all.proxy_arp=0
~~~

3. Gateway의 Redirect Packet 차단, 스푸핑을 방어.
~~~
net.ipv4.conf.eth0.secure_redirects=0
net.ipv4.conf.lo.secure_redirects=0
net.ipv4.conf.default.secure_redirects=0
net.ipv4.conf.all.secure_redirects=0
~~~

4. 스푸핑을 막기 위해 source route 패킷을 차단.
- 소스 라우팅을 허용할 경우 악의적인 공격자가 소스 라우팅을 사용할 수 있다. 이러한 소스 라우팅이 가능한 것을 이용해 공격자가 마치 신뢰받는 호스트나 클라이언트인 것처럼 위장할 수 있는 것이다.
~~~
net.ipv4.conf.eth0.accept_source_route=0
net.ipv4.conf.lo.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0
net.ipv4.conf.all.accept_source_route=0
~~~

3. Broadcast로부터 오는 핑을 차단(Smurf 공격 방어).
~~~
net.ipv4.icmp_echo_ignore_broadcasts=1
~~~

4. IP 나 TCP 헤더가 깨진 bad icmp packet을 무시한다.
~~~
net.ipv4.icmp_ignore_bogus_error_responses = 1
~~~

5. 자신의 네트워크가 스푸핑된 공격지의 소스로 쓰이는 것을 차단.
- 모든 인터페이스에서 들어오는 패킷에 대해 reply를 하여 들어오는 인터페이스로 나가지 못하는 패킷을 거부한다.
~~~
net.ipv4.conf.eth0.rp_filter=2
net.ipv4.conf.lo.rp_filter=2
net.ipv4.conf.default.rp_filter=2
net.ipv4.conf.all.rp_filter=2
~~~

6. bootp 패킷을 허용하지 않는다.
~~~
net.ipv4.conf.eth0.bootp_relay=0
net.ipv4.conf.lo.bootp_relay=0
net.ipv4.conf.default.bootp_relay=0
net.ipv4.conf.all.bootp_relay=0
~~~

7. 스푸핑된 패킷이나 소스라우팅, Redirect 패킷에 대해 로그파일에 정보를 남긴다.
~~~
net.ipv4.conf.eth0.log_martians=1
net.ipv4.conf.lo.log_martians=1
net.ipv4.conf.default.log_martians=1
net.ipv4.conf.all.log_martians=1
~~~

8. 1/100초에 받아들이는 igmp "memberships"의 수
~~~
net.ipv4.igmp_max_memberships=1
~~~

9. Linux Default TTL (Time To Live) 설정.
- default(64) 적당함.
- 네트워크 복잡도 높은 사이트에서 조정 가능하나 장애 가능성 높음.
~~~
net.ipv4.ip_default_ttl=64
~~~

10. 패킷을 포워딩 차단.
- 가상화 환경(Hypervisor,VM,Docker) 에서는 차단시 장애발생.
- 반드시 물리서버 환경에서만 적용.
~~~
net.ipv4.ip_forward=0
~~~

11. fragmented packet이 메모리에 존재하는 시간을 설정 : 15초
~~~
net.ipv4.ipfrag_time=15
~~~

12. SYN_Flooding 공격 대비로 백로그큐(Backlog Queue)가 가득차면 다른 접속 요구를 거절됨.
~~~
net.ipv4.tcp_max_syn_backlog = 1024
~~~

13. TCP 연결에서 Three-way Handshake가 성공적으로 이루어지지 않으면 더 이상 소스 경로를 거슬러 올라가지 않도록한다. 따라서 적절한 연결 요청에 대해서만 연결을 맺는다.
~~~
net.ipv4.tcp_syncookies = 1
~~~
- (참고) syncookies가 작동할 때 SYN Flooding 공격이 있으면 messages 파일에 아래와 같은 내용이 출력된다.
~~~
 possible SYN flooding on port 80. Sending cookies.
~~~

14. 일정한 시간과 IP별로 보내고 받는 SYN 재시도 횟수를 3회로 제한한다.
- 이 옵션은 스푸핑된(위조된) 주소로 오는 SYN 연결의 양을 줄여준다.
- 기본 값은 5(180 초에 대응)이며 255를 넘지 않아야 한다.
~~~
net.ipv4.tcp_syn_retries = 3
~~~

15. passive TCP 접속시도가 재접속을 하기 위한 SYNACKs의 값을 정한다. 255 보다 높게 지정할 수 없다. 기본값은 5이며, 180초에 대응이 된다.
~~~
net.ipv4.tcp_synack_retries = 3
~~~

16. TCP 연결을 위해 재시도 할 횟수, 최소 값과 기본 값은 3이다.
~~~
net.ipv4.tcp_retries1=3
~~~

17. TCP 연결을 끊기 전에 재시도할 횟수.
~~~
net.ipv4.tcp_retries2=7
~~~

18. 연결 종료시 소요되는 시간을 줄여준다(Default=60).
~~~
net.ipv4.tcp_fin_timeout=20
~~~

19. 동시 유지 가능한 TIMEWAIT 소켓의 수.
- 만약 지정된 숫자를 초과하였을 경우에는 timewait 소켓이 없어지며 경고 메시지가 출력된다.
- 단순 DoS 공격을 차단 가능, 임의로 이 값을 줄여서는 안된다.
- 메모리가 충분하다면 적절하게 늘려주는 것이 좋은데, 64M 마다 180000으로 설정하면 된다.

~~~
 # 64M -> 180000
 # 128M -> 360000
 # 256M -> 720000
 # 512M -> 1440000
 # 1G -> 2880000
 # 2G -> 5760000
#net.ipv4.tcp_max_tw_buckets = 180000
~~~

20. 연결이 끊어졌다고 판단할 때까지, 얼마나 keepalive probe 를 보낼지 결정. 기본값 9회 간단한 DoS 공격 차단.
~~~
net.ipv4.tcp_keepalive_probes=2
~~~

21. keepalive 활성화 경우, 얼마나 자주 TCP 가 keepalive 메세지를 보내게 할 것인지를 설정.
~~~
net.ipv4.tcp_keepalive_time=30
~~~

22. keepalive_probes 를 보낼 간격을 정함. probe 를 보낸 후, probes * intvl 의 시간이 지나도록 응답이 없으면 연결이 해제된 것으로 간주하게 됨. 기본 값의 사용시 11분 15초 동안 재시도를 하고 연결을 취소함. (단위 : Second)
~~~
net.ipv4.tcp_keepalive_intvl = 10
~~~

23. 서버 쪽에서 닫은 TCP 연결을 끊기 전에 확인하는 횟수를 정한다. 기본 값은 7 로 RTO 50 초에서 16 분 사이에 해당한다. (웹 서버 운영 중 해당 값을 줄여서 세션 리소스의 낭비를 방지할 수 있다)
~~~
net.ipv4.tcp_orphan_retries = 2
~~~

24. SYN 패킷을 전송한 후에 로스가 발생을 하여 ACK 를 일부 받지 못했을 경우, 선택적으로 (selected) 받지못한 ACK 만 받도록 요청하는 것을 허락한다. 로스가 많은 네트워크에서는 상당히 중요한 역할을 한다.
~~~
net.ipv4.tcp_sack = 1
~~~
 
 
 
 
