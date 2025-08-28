---
layout: post
title:  "CIQ Product Onboarding."
date:   2025-08-27 22:47:36 +0900
categories: [Linux, System]
---

- [1. CIQ Depot Client](#1-ciq-depot-client)

### 1. CIQ Depot Client

* CIQ의 모든 소프트웨어 및 리소스를 액세스를 제공할수 하는 컨텐츠 전송 플랫폼.
* CIQ Depot Client **(호스트에 설치 하여 저장소를 쉽게 인증하고 액세스 할수 있는 도구)**로 구성된다.
* Depot Client 설치 방법 

> **1.수동 설치 방법 (Manually install)**
> ```bash
> dnf install -y https://depot.ciq.com/public/files/depot-client/depot/depot.x86_64.rpm
> ```

>**2.Client Host 셋팅 및 삭제**
>* CIQ Depot Client가 정상적으로 설치 되었으면 아래와 같이 사용자를 CIQ Depot 서비스에 등록한다.
>* Login 정보를 삭제하기 위해서는 login.yaml 파일을 삭제 한다.
>```bash
> sudo depot login -U [USER_STRING] -t [USER TOKEN]
> sudo rm -f /root/.depot/client.yaml
>```
> * USER_STRING과 USER TOKEN은 등록된 CIQ 포탈에서 확인한다.
> * 호스트가 등록되면 해당 호스트에 대한 DNF를 구성할수 있다. 기본적으로 활성화된 모든 저장소가 자동으로 구성된다. 

>**3.Client Host 등록확인**
>```bash
> sudo depot list 
>```
>* 아래와 같이 CIQ의 등록된 제품 리소스가 표시되면 정상
<div style="clear:both;"></div>
> <img src="/img/linuxtech/depotlist.PNG" alt="screenshot" align=left width="850"/>
<div style="clear:both;"></div>

>**4.리포지터리 활성화**
> * 등록된 제품에 대한 Repository를 자동으로 활성화 (Enable) 시킨다.
> * 반대로 등록된 제품에 대한 저장소를 비활성화 (Disable) 하면 자동으로 저장소 DNF는 삭제된다.
>```bash
> sudo depot enable PRODUCT_ID 
> sudo depot disable PRODUCT_ID
>```
>* 아래와 같이 Enable된 제품에 대한 저장소의 DNF가 자동으로 생성된다. 
>* /etc/yum.repos.d/depot-rlc-9.6.repo 저장소 파일이 자동으로 생성된 것을 확인할수 있다.
<div style="clear:both;"></div>
> <img src="/img/linuxtech/depotenable.PNG" alt="screenshot" align=left width="850"/>
<div style="clear:both;"></div>

>**5.개별 rpm 파일 다운로드**
> * Depot client를 이용해서 개별적인 rpm 패키지 파일을 다운로드 받을수 있다.
>```bash
> depot download [Package-name]
> depot download -u https://{user}:{token}@depot.ciq.com/download/lts-9.4/rocky-baseos-9.4.x86_64/Packages/a/accel-config-4.1.3-2.> el9.i686.rpm
> depot download -p lts-9.4 -r rocky-baseos-9.4.x86_64 -f Packages/a/accel-config-4.1.3-2.el9.i686.rpm
> depot download -p lts-9.4 -r rocky-baseos-9.4.x86_64 -f Packages/a/accel-config-4.1.3-2.el9.i686.rpm -c 5
>```
> * depot을 이용한 개별 파일에 대한 다운로드 방법은 **depot download -h** 명령어를 통해서 자세하게 확인할수 있다.
<div style="clear:both;"></div>
><img src="/img/linuxtech/downloadrpm.PNG" alt="screenshot" align=left height=450 width="850"/>
<div style="clear:both;"></div>
