---
layout: post
title: AIX OS의 /dev/random과 /dev/urandom 차이점
tag:  AIX
subtitle: AIX WAS 설정 시 참고자료
---


# IBM AIX OS의 /dev/random 과 /dev/urandom 차이점

/dev/random 및 /dev/urandom  chractor 디바이스는  인터럽트 타이밍 또는 디바이스에 기록 된 입력으로부터 생성 된 암호학적으로 안전한 랜덤 출력을 제공합니다.

/dev/random 디바이스는 고품질의 암호의 보안 랜덤  출력을 제공하기 위한 것이며 출력 생성 시 충분한 (동일한 양 또는 더 많은 양의) 랜덤 입력을 사용할 수 있는 출력 만 반환합니다.

만약 랜덤 입력이 충분하지 않은데 디바이스가 열렸을 경우 O_NONBLOCK 플래그가 지정되어 있지 않으면 요청을 처리 할 수있을 때까지 /dev/random 디바이스의 읽기가 차단됩니다. 이 경우 생성 될 수있는 많은 양의 출력이 반환됩니다 ( 오류 코드 EAGAIN. ) 
/dev/urandom 디바이스는 신뢰할 수 있는 랜덤 출력 소스를 제공하지만 랜덤 입력이 불충분 할 경우, 랜덤 출력이 생성되지 않을 것 입니다. 

~~~
위와 같은 이유로 WAS의 JAVA 옵션을 설정시 default 값인 /dev/random이 아닌 /dev/urandom을 사용합니다.
~~~

/dev/urandom 디바이스는 읽기는 항상 차단하지 않고 요청 된 출력의 양을 반환합니다. 랜덤 입력이 가능한 경우, 대체 입력은 난수 생성기에 의해 처리되어 암호 보안 출력을 제공하며, 그 강도는 난수 생성기에서 사용되는 알고리즘의 강도를 반영합니다.

랜덤 입력 없이 생성 된 출력은 이론적으로 랜덤 입력에서 생성 된 출력보다 안전하지 않으므로 출력의 보안에 대한 높은 수준의 신뢰가 요구되는 응용 프로그램에 대해서는 /dev/random을 사용해야 합니다.

~~~
즉 굉장히 높은 보안이 요구되는 비지니스 로직을 가진 서비스에서는 /dev/random 의 사용을 권장
~~~

두 디바이스 중 하나에 기록 된 데이터는 저장된 임의 입력의 풀에 추가되며 출력을 생성하는 데 사용될 수 있습니다. 쓰기는 두 디바이스에서 동일하게 작동하며 차단되지 않습니다.


# 구현 세부 사항

/dev/random 및 /dev/urandom 디바이스는 난수 생성기가 로드 될 때 디바이스 구성 하위 시스템에서 할당 한 주 번호와 부 번호로 만들어지므로 디바이스를 찾거나 열려고 할 때 디바이스 이름을 항상 사용해야 합니다. 디바이스는 난수 생성기가 언로드되면 삭제됩니다.

shutdown 명령을 사용하여 시스템을 종료하면 /dev/urandom 디바이스에서 출력을 가져와 다음 부팅시 난수 생성기가 로드 될 때 /dev/random 디바이스에 재기록되어 생성기에 시작 엔트로피를 제공합니다. 부팅 후 저장된 랜덤 입력의 품질을 향상시킵니다.

저장된 랜덤 입력의 풀이 절반 이하로 떨어지면 인터럽트 타이밍에서 수집되고 풀이 다시 채워질 때까지 계속 수집됩니다. 이 프로세스는 타이밍이 수집되는 동안 모든 외부 인터럽트에 약간의 성능 영향을 미치며 타이밍이 수집되지 않으면 중단됩니다.

랜덤 디바이스 중 하나에 기록 된 데이터는 또한 저장된 랜덤 입력 풀에 기여하며 출력에 영향을 미칠 수 있으므로 이러한 디바이스에 대한 쓰기 작업 권한 있어야 합니다. 이것은 디바이스의 사용 권한에 의해 동작하므로 원하는 경우 관리자가 완전히 변경할 수 없도록 수정 할 수 있습니다.


<hr>
[ 참고 ] AIX 환경 아래  /dev/random 사용 중 발생한 장애 리포트 및 커뮤니티

[1] [PI52301: REDUCE READS TO /DEV/RANDOM CAUSING CSFSERV CSFRNG ACCESS](http://www-01.ibm.com/support/docview.wss?mhq=%2Fdev%2Frandom%20performance&mhsrc=ibmsearch_a&uid=isg1PI52301)

[2] [ADD NEW SECURERANDOM FUNCATIONALITY FOR IBMJCE AND IBMSECURERAND OM PROVIDERS](http://www-01.ibm.com/support/docview.wss?uid=swg1IV75079)

[3] [AIX WebLogic Server 성능 저하 리포트](https://blog.naver.com/solvage/220312787411)

[참고] [/dev/random , /dev/urandom 테스트 방법](https://blog.csdn.net/zqy2000zqy/article/details/1154842)

[출처] AIX 6.1 File Referance 