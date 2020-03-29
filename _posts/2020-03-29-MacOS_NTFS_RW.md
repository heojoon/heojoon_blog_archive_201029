---
layout: post
title: MacOSX_NTFS_HDD_Write_방법
tag: MacOSX
subtitle: MacOSX_NTFS_HDD_Write_방법
---

#  Mac OS X 의 NTFS HDD 읽기 쓰기 

MacBook 에서 윈도우에서 쓰던 외장하드를 붙였더니 읽기 전용으로 나온다.

그래서 구글링을 해보니 읽기 전용으로만 마운트가 된다고 한다. 

기본 패키지 프로그램인 Mount-nfts 나 mount -t ntfs -o noauto,sosuid -w  같은 명령들은 다 읽기전용으로 마운트 된다.

이를 해결하기 위해서는 2가 방법이 있다.

1. Mac OS 용 Paragon Software (유료) 
2. 오픈소스 NTFS-3G (무료)


역시 오픈소스 NTFS-3G로 진행한다.  home brew 가 설치되어 있으면 간단히 진행 할 수 있다.

~~~
$ brew cake install osxfuse
$ brew install ntfs-3g
~~~

diskutil list 명령어로 NTFS 파일시스템으로 표시되는 디스크를 확인한다.

~~~
$ diskutil list
~~~

다음은 디스크를 마운트하고자 하는 디렉토리를 생성 한다.

~~~
$ sudo mkdir /Volumes/ExtraHDD
~~~

ntfs-3g 명령어로 NTFS 디스크를 마운트 한다.

~~~
$ sudo /usr/local/bin/ntfs-3g /dev/disk2s1 /Volumes/ExtraHDD -olocal -oallow_other
~~~



위와 같이 마운트 명령어를 내리면 보안 및 개인 정보 보호에서 OSXFUSE 앱을 허용 할 것 이냐고 물어보면 "확인" 버튼을 누르고 시스템 환경 설정에서 보안 및 개인 정보 보호에서 일반 > 다운로드한 앱 허용 > App Store  및 확인된 개발자 에서 OSXFUSE 앱을 허용한다.  



그러면 아래와 같이 바탕화면에 마운트 한 디스크가 보인다. 이 디스크는 읽기 및 쓰기가 가능해졌다!!


[참고] 

* <a href="https://blog.fosketts.net/2010/11/29/write-windows-ntfs-drive-mac-os-106-snow-leopard/" MacOSX ntfs 마운트 방법 2가지>
* <a href="https://blog.kylehuynh.com/mount-ntfs-drive-with-readwrite-mode-in-mac-os/" MacOSX home brew로 ntfs-3g 설치하기>

