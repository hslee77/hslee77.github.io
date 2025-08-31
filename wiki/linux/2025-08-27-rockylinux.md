---
layout: post
title:  "CIQ Product Onboarding."
date:   2025-08-27 22:47:36 +0900
categories: [Linux, System]
---

- [1. CIQ Depot Client](#1-ciq-depot-client)
- [2. Rsync를 사용하여 CIQ 저장소 미러링](#2-rsync를-사용하여-ciq-저장소-미러링)
- [3. Reposync를 이용한 CIQ 저장소 미러링](#3-reposync를-이용한-ciq-저장소-미러링)

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

### 3. Reposync를 이용한 CIQ 저장소 미러링

* reposync는 원격 저장소를 로컬 복사본으로 생성하는 DNF 유틸리티
* reposync를 구성하고 CIQ 저장소를 로컬로 미러링 하는 방법을 사용을 제공


> **1.reposync 동작 방식**

> * reposync 명령어는 다음과 같은 기능을 수행합니다:
> *원격 저장소의 로컬 복사본을 생성
> *로컬 디렉토리에 이미 존재하는 패키지는 다시 다운로드하지 않음
> *모든 활성화된 저장소 또는 특정 저장소만 동기화 가능
> * **--repo, --enable-repo, --disable-repo** 와 같은 표준 DNF 옵션 지원
>
> **2.reposync 설정하기**
> **1. 리포지토리 설정 파일 생성**
>
> * CIQ 포털에서 My Products로 이동
> * 미러링하려는 리포지토리가 포함된 제품을 클릭
> * 해당 리포지토리를 클릭하여 상세 정보 열기
> * DNF Repo Config 옵션을 찾아 클릭
> * 제공된 설정 내용을 복사
> * /etc/yum.repos.d/ 디렉토리에 설명이 담긴 이름(예: ciq-bridge.repo)으로 새 파일 생성
복사한 설정 내용을 해당 파일에 붙여 넣고 저장
>
> * 리포터리 설정 파일 예제 (Example)
```bash
[ciq-bridge.x86_64]
name = CentOS 7.9 CIQ Bridge Updates (x86_64)
baseurl = https://depot.ciq.com/files/bridge/ciq-bridge.x86_64
gpgkey = https://ciq.com/keys/rpm-gpg-key-ciq
username = DEPOT_USER
password = DEPOT_TOKEN
metadata_expire = 5
priority = 50
repo_gpgcheck = false
gpgcheck = true
enabled = true
skip_if_unavailable = true
```
**주요 항목 설명**
>
> * **[ciq-bridge.x86_64]** : 리포지토리의 ID (고유 식별자)
> * **name** : 리포지토리 이름 (사람이 읽기 쉽게 표시됨)
> * **baseurl** : 패키지를 다운로드할 원격 저장소 URL
> * **gpgkey** : 패키지 검증에 사용할 GPG 키 위치
> * **username** / password : 인증을 위한 사용자 이름과 토큰(DEPOT 포털에서 제공)
> * **metadata_expire** : 메타데이터 캐시 만료 시간 (단위: 분, 초 등 상황에 따라 다름)
> * **priority** : 저장소 우선순위 (낮을수록 높은 우선순위)
> * **repo_gpgcheck** : 리포지토리 메타데이터 서명 검증 여부 (false = 비활성화)
> * **gpgcheck** : 패키지 서명 검증 여부 (true = 활성화)
> * **enabled** : 리포지토리 사용 여부 (true = 활성화)
> * **skip_if_unavailable** : 리포지토리에 접근할 수 없을 때 건너뛸지 여부

> 2. **인증 설정 (Configure Authentication)**
>
> * DEPOT_USER와 DEPOT_TOKEN 값을 포털의 Access Token 페이지에서 발급받은 실제 자격 증명으로 교체.
대체 인증 방법: 자격 증명을 baseurl에 직접 포함할 수도 있다.
```bash
baseurl = https://DEPOT_USER:DEPOT_TOKEN@depot.ciq.com/files/bridge/ciq-bridge.x86_64
```
> * 여러 리포지토리를 동시에 동기화할 때 유용gka
>
> **3. reposync 실행 (Run reposync)**
> * 설정 파일이 준비되면 아래 명령어로 리포지토리를 동기화합니다:
```bash
dnf reposync --repoid=ciq-bridge.x86_64 --download-metadata --download-path=/var/www/html/repos/
```
> * ciq-bridge.x86_64 → repo 파일에서 정의된 리포지토리 ID로 교체
> * /var/www/html/repos/ → 원하는 저장 경로로 교체
>
> **고급 옵션 (Advanced Options)**
>
> * 1) 다중 리포지토리 동기화 (Syncing Multiple Repositories)
> * 모든 활성화된 리포지토리를 동기화하려면:
```bash
dnf reposync --download-metadata --download-path=/var/www/html/repos/
```
> 2) 리포지토리 메타데이터 생성 (Creating Repository Metadata)
> 동기화 후 메타데이터를 생성해야 할 수 있다.
```bash
createrepo_c /var/www/html/repos/ciq-bridge.x86_64/
```
> 3) 정기 동기화 스케줄링 (Scheduling Regular Syncs)
미러를 최신 상태로 유지하려면 cron을 사용해 스케줄링합니다.

> **동기화 스크립트 작성:**
```bash
#!/bin/bash
dnf reposync --repoid=ciq-bridge.x86_64 --download-metadata --download-path=/var/www/html/repos/ >> /var/log/reposync.log 2>&1
createrepo_c --update /var/www/html/repos/ciq-bridge.x86_64/
```
> **크론 작업에 등록 (매일 새벽 2시 실행):**
```bash
0 2 * * * /path/to/your/reposync-script.sh
```
> ### 문제 해결 (Troubleshooting)
>
> **일반적인 문제 및 해결 방법**
>
> * **인증 오류 (Authentication Errors)**: repo 설정 파일에서 자격 증명이 올바른지 확인
> * **패키지 누락 (Missing Packages)**: 리포지토리가 올바르게 활성화되어 있는지 확인
> **권한 거부 (Permission Denied)**: 대상 디렉토리에 쓰기 권한이 있는지 확인
>
> ### 다음 단계 (Next Steps)
>
> 로컬 미러를 생성한 후 추가적으로 필요한 작업:
> * 웹 서버를 설정하여 리포지토리를 제공
> * 시스템을 업데이트하여 로컬 미러를 사용하도록 구성
> * 정기 동기화 스케줄을 설정하여 최신 상태 유지
>
### 추가 지원 (Additional Assistance)
>
> * 추가 지원이 필요할 경우, CIQ 지원팀에 문의하십시오:
> **[support@ciq.com](mailto:support@ciq.com)**