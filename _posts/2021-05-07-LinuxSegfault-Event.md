---
layout: post
title:  "What is Segfault Message?"
date:   2021-05-07 12:07:12 +0530
categories: Linux,cloud
---
 
Linux에서 세그먼테이션 오류 (segfault) 또는 액세스 위반은 운영 체제 (OS)에 **메모리 액세스 위반**을 알리기 위해 메모리 보호 기능이있는 하드웨어에서 발생하는 오류입니다.  

리눅스 커널은 문제가 발생하기 되면 일반적으로 프로세스 11번과 같은 신호를 보내서 문제의 프로세스에 결함이 있음을 전달함으로써 이러한 문제에 대응하게 됩니다.

일부의 경우에는 사용자가 직접 문제의 프로세스가 발생했을 경우 이러한 signal을 수신하고 처리 할수 있도록 customer handler를 설치하여 사용하는 경우도 있으나.  

그렇지 않을 경우에는 Linux의 커널에서 동작하고 있는 **process signal** 처리기를 사용할수 있습니다.

segfault는 일반적으로 프로세스를 종료하고 적절한 ulimit 설정을 갖춘 코어 덤프를 생성합니다.
## Segfaul의 유형은?? 

```javascript
kernel: login[118125]: segfault at 0 ip 00007f4e4d5334a8 sp 00007fffe9177d60 error 15 in pam_unity_uac.so[7f4e4d530000+b000]
kernel: crond[16398]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000]
kernel: crond[17719]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000
kernel: login[118125]: segfault at 0 ip 00007f4e4d5334a8 sp 00007fffe9177d60 error 15 in pam_unity_uac.so[7f4e4d530000+b000]
kernel: crond[16398]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000]
kernel: crond[17719]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f500
```

Segmentation faults 의 경우 다양한 원인이 존재하지만 일반적인 경우 **C 언어로 작성된 프로그램에서 공통적으로 발생** 하는데 이러한 프로그램들이 동작할때 가상 메모리 주소 지정, 특히 불법적인 액세스를위한 포인터 사용시 segfault 에러가 발생하게 된다.

또 다른 유형의 메모리 액세스 오류는 여러 가지 원인이있는 메모리 버스 오류가 있지만 오늘날에는 거의 발생하지 않는다.

이는 주로 잘못된 물리적 메모리 어드레싱 또는 정렬되지 않은 메모리 액세스로 인해 발생하는 경우인데 이러한 메모리 참조는 프로세스가 허용되지 않는 것이 아니라,
하드웨어가 주소를 다룰 수 없다는 것을 참조한다, 즉 하드웨어에서 발생하는 에러가 되는 것이다.


## 어떻게 확인하나? 

1. Signify segfault

일반적으로 segfault는 특정 프로세스나 프로그램에서 오류를 나타냅니다. 즉 이러한 에러가 리눅스 커널의 에러를 의미하지는 않습니다.

리눅스 커널의 경우 아래와 같이 특정 프로세스 또는 프로그램에서 발생하는 오류를 감지하고 로그를 기록하는 역활을 하는 것입니다.

```javascript
kernel: login[118125]: segfault at 0 ip 00007f4e4d5334a8 sp 00007fffe9177d60 error 15 in pam_unity_uac.so[7f4e4d530000+b000]
kernel: crond[16398]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000]
kernel: crond[17719]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000
kernel: login[118125]: segfault at 0 ip 00007f4e4d5334a8 sp 00007fffe9177d60 error 15 in pam_unity_uac.so[7f4e4d530000+b000]
kernel: crond[16398]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000]
kernel: crond[17719]: segfault at 14 ip 00007fd612c128f2 sp 00007fff6a689010 error 4 in pam_seos.so[7fd612baf000+f5000

```

2. Error 메세지가 의미하는 것은?
RIP값은 명령 포인터 레지스터 값이며, RSP는 스택 포인터 레지스터 값입니다. 오류 값은 페이지 오류 오류 코드 비트의 비트 마스크입니다 (from arch/x86/mm/fault.c):

```javascript
* bit 0 == 0: no page found 1: protection fault

* bit 1 == 0: read access 1: write access

* bit 2 == 0: kernel-mode access 1: user-mode access

* bit 3 == 1: use of reserved bit detected

* bit 4 == 1: fault was an instruction fetch

아래는 segfault에 대한 error 비트을 정의한 것입니다.

enum x86_pf_error_code {

PF_PROT = 1 << 0,

PF_WRITE = 1 << 1,

PF_USER = 1 << 2,

PF_RSVD = 1 << 3,

PF_INSTR = 1 << 4,

};

```

예를 들어서 십진수 에러코드 15가 출력되면 2진수로 아래와 같이 01111로 표현하게 된다. 
```javascript
01111
^^^^^
||||+---> bit 0
|||+----> bit 1
||+-----> bit 2
|+------> bit 3
+-------> bit 4
```
10진수에서 2진수로 변환된 에러코드를 RIP 값에 역순으로 대입을해보면 이 메시지는 **_응용 프로그램이 사용자 모드에서 메모리의 예약 된 섹션에 대한 액세스를 쓰려고 시도했기 때문에_** 응용 프로그램이 보호 오류를 트리거했음으로 해석될수 있습니다.

[참고코드] : [segfault]


[segfault]: https://kernel.googlesource.com/pub/scm/linux/kernel/git/ralf/linux/+/linux-2.5.22/arch/x86_64/mm/fault.c