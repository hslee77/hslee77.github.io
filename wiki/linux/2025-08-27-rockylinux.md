---
layout: post
title:  "CIQ Product Onboarding."
date:   2025-08-27 22:47:36 +0900
categories: [Linux, System]
---

- [1. CIQ Depot Client](#1-ciq-depot-client)
- [2. Rsync를 사용하여 CIQ 저장소 미러링](#2-rsync를-사용하여-ciq-저장소-미러링)

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
> depot download -p lts-9.4 -r rocky-baseos-9.4.x86_64 -f Packages/a/accel-config-4.1.3-2.elfa9.i686.rpm
> depot download -p lts-9.4 -r rocky-baseos-9.4.x86_64 -f Packages/a/accel-config-4.1.3-2.el9.i686.rpm -c 5
>```
> * depot을 이용한 개별 파일에 대한 다운로드 방법은 **depot download -h** 명령어를 통해서 자세하게 확인할수 있다.
<div style="clear:both;"></div>
><img src="/img/linuxtech/downloadrpm.PNG" alt="screenshot" align=left height=450 width="850"/>
<div style="clear:both;"></div>

### 2. Rsync를 사용하여 CIQ 저장소 미러링

* CIQ 저장소를 Rsync를 이용하여 로컬 스토리지로 미러링하는 방법을 제공한다. 
* rsync URI를 찾아 로컬 미러를 생성하는 방법을 설명한다. 

> **1.스토리지 용량**
> * CIQ 브릿지 저장소는 대략 **1.4GB** 
> * 전체 CentOS 저장소의 경우 **100GB** 
> * 저장소를 동기화 하기 전에 반드시 스토리지 용량에 대한 점검을 진행할것을 권고 
> 
> **2.rsync URI 찾는 방법**
> * CIQ 포털에서 My Product로 이동한다.
> * 미러링하려는 저장소가 포함되어 있는 제품을 클릭한다. 
> * Product view 에서 원하는 저장소를 찾는다. 
> * 저장소를 클릭하여 세부 정보를 연다. 
> * 저장소 정보에서 Rsync URI 필드를 찾는다. 
> 
> **3.기본 Rsync 명령어** 
> * 로컬 저장소로 미러링 하기 위해서는 아래와 같은 기본 명령어를 사용한다. 
>```bash
> RSYNC_PASSWORD=[access token] rsync -avSHP [rsync_uri] [local_directory]
>```
>
> * 기본 명령어에 대한 예제
>```bash
> RSYNC_PASSWORD=[access token] rsync rsync://[userName]@rsync.depot.ciq.com/lts-9.2:rocky-baseos-9.2.x86_64
>```
>
> **4.cron 을 이용한 미러링 스케쥴링**
>* rsync 미러링 스크립트 작성
>```bash
> #!/bin/sh 
> rsync -avSHP --delete [rsync_uri] [local_directory] >> /var/log/rsync-repo.log 2>&1
>```
>
>* cron job 스케쥴러에 등록
>```bash
> 0 2 \*\*\* /path/to/your/sync-script.sh
>```
> 
>* 트러블 슈팅 (Troubleshooting)
>* 만약 제품 Repsository에 Rsync URI 옵션이 포함되어 있지 않다면 rsync 미러링이 활성화 되어 있지 않았을 가능성이 있음.
>* 제품 Repository가 reposync를 지원하는지 확인 
>* CIQ 서포트를 통해서 지원을 받아야함 (support@ciq.com)