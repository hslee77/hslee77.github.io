---
layout: post
title:  "리눅스 네트워크부터 연결하자"
date:   2025-07-24 15:10:36 +0900
categories: [Linux, Network]
---
리눅스를 활용하기 위해서는 전체적인 구조를 이해하는 것도 중요하지만 운영에 필요한 명령어들을 익히고 활용하는 것도 무엇보다 중요하다 물론 클라우드 시대가 되면서 Instance라는 개념으로 Linux가 운영되면서 이전보다는 Deep Dive한 개념을 들여다 보는 일들이 많지 않으나 그래도 모든 플랫폼의 기본이 되는 환경은 리눅스라는 점에서는 운영에 필요한 역량은 반드시 알아둘 필요가 있지 않을까 싶다. 

## 1.네트워크 연결설정

## nmcli로 네트워크 연결하기

**`Red Hat Enterprise Linux 8(RHEL 8)`** 에서 네트워크를 설정하고 연결하는 방법은 기본적으로 **`nmcli(NetworkManager CLI)`** 명령어를 사용하거나, 필요에 따라 설정 파일을 직접 편집하는 방식으로 수행된다.

### 1.1 Active 인터페이스 찾기

```javascript
nmcli device status

DEVICE   TYPE      STATE      CONNECTION
ens33    ethernet  connected  Wired_connection_1
lo       loopback  unmanaged  --
```
### 1.2 네트워크 연결 생성 (고정ip)

```javascript
nmcli connection add type ethernet con-name my-ethernet ifname ens33 ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
```
- **`type ethernet`** : 이더넷 연결 유형.
- **`con-name`** : 연결 이름 지정.
- **`ifname`** : 네트워크 인터페이스 이름(예: ens33).
- **`ipv4.addresses`** : IP 주소와 서브넷 마스크(예: 192.168.1.100/24).
- **`ipv4.gateway`** : 기본 게이트웨이.
- **`ipv4.dns`** : DNS 서버.
- **`ipv4.method manual`** : 고정 IP 설정(자동 할당은 auto).

### 1.3 네트워크 연결 활성화

```bash
nmcli connection up my-ethernet [지정된 연결이름]
```
### 1.4 네트워크 연결 비활성화
```bash
nmcli connection down my-ethernet [지정된 연결이름]
```
### 1.5 네트워크 연결 DHCP
```bash
nmcli connection modify my-ethernet ipv4.method auto
nmcli connection up my-ethernet
```

### 1.6 네트워크 설정파일 편집 

네트워크 설정은 **`/etc/sysconfig/network-scripts/ifcfg-<인터페이스 이름>`** 파일에서 관리된다. 

### 1.7 네트워크 설정 파일 

해당 문서에서는 ens33 인터페이스를 가정하여 작성한다.

```bash
/etc/sysconfig/network-scripts/ifcfg-ens33

TYPE=Ethernet
BOOTPROTO=none
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8

dhcp로 설정할 경우 

TYPE=Ethernet
BOOTPROTO=dhcp
NAME=ens33
DEVICE=ens33
ONBOOT=yes
```

수정된 네트워크 파일을 반영하기 위해서는 네트워크 데몬 재시작
```bash 
nmcli connection reload
systemctl restart NetworkManager
```

## 1.8 연결상태 확인 및 점검 

```bash 
ip addr show
nmcli general status
ping -c 4 8.8.8.8
```
---
## 2. Bonding 구성하기

RHEL 8에서 **`이더넷 본딩(또는 링크 어그리게이션)`** 은 여러 네트워크 인터페이스를 하나의 논리 인터페이스로 묶어 네트워크 성능을 향상시키거나 고가용성을 제공하는 데 사용됩니다.
RHEL 8에서는 nmcli 명령어와 NetworkManager를 사용하여 본딩 구성을 간편하게 수행할 수 있습니다.


![](https://velog.velcdn.com/images/kainos/post/954504f0-f988-4fd4-ba22-0a566386f420/image.png)
[출처 : [레드햇 문서](https://docs.redhat.com/ko/documentation/red_hat_virtualization/4.1/html/technical_reference/multiple_bridge_multiple_vlan_and_bond_configuration#Multiple_Bridge_Multiple_VLAN_and_Bond_Configuration)]

### 2.1 bonding 구성하기전 

사용 가능한 인터페이스 찾기. 

```bash
nmcli device status

DEVICE   TYPE      STATE      CONNECTION
ens33    ethernet  connected  Wired_connection_1
ens34    ethernet  connected  Wired_connection_1
lo       loopback  unmanaged  --
```
본딩을 구성하기 위한 ens33, ens34 인터페이스를 확인한다. 

### 2.2 bonding 모드 이해 

본딩 모드 이해 본딩 모드는 네트워크 트래픽을 처리하는 방식에 따라 다릅니다.

```bash
- mode=0 (balance-rr) : 라운드 로빈 방식(트래픽 부하 분산).
- mode=1 (active-backup) : 활성-대기 방식(고가용성 제공).
- mode=2 (balance-xor) : XOR 방식(특정 알고리즘 기반 트래픽 분산).
- mode=4 (802.3ad) : LACP(Link Aggregation Control Protocol) 사용. / 스위치에서 지원가능해야 함.
- mode=5 (balance-tlb) : 송신 트래픽만 부하 분산.
- mode=6 (balance-alb) : 송신 및 수신 트래픽 부하 분산.
```
### 2.3 nmcli를 이용한 bonding 구성 

```bash
nmcli connection add type bond con-name bond0 ifname bond0 mode active-backup
```
**주요 옵션** :
- **`type bond`** : 본딩 인터페이스 생성.
- **`con-name bond0` **: 본딩 연결 이름 지정.
- **`ifname bond0`** : 본딩 인터페이스 이름.
- **`mode active-backup`**: 본딩 모드 설정(예: active-backup).

### 2.4 slave 인터페이스 추가 

bond0 인터페이스에 포함될 물리 네트워크 인터페이스를 슬레이브로 설정한다.

```bash
nmcli connection add type ethernet con-name ens33-slave ifname ens33 master bond0 [첫번째 인터페이스]

nmcli connection add type ethernet con-name ens34-slave ifname ens34 master bond0 [두번째 인터페이스]
```

### 2.5 bond0 인터페이스에 ip추가 

```bash
nmcli connection modify bond0 ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
```
### 2.6 bond0 인터페이스 dhcp 설정 

```bash
nmcli connection modify bond0 ipv4.method auto
```
### 2.7 bond0 인터페이스 활성화 

bonding 인터페이스가 구성이 완료되면 모든 물리 인터페이스 까지 활성화 한다.

```bash
nmcli connection up bond0
nmcli connection up ens33-slave
nmcli connection up ens34-slave
```

### 2.8 본딩 인터페이스 파일 생성

**`/etc/sysconfig/networ-scripts/`** 경로안에 네트워크 인터페이스 파일 생성.

```bash
/etc/sysconfig/network-scripts/ifcfg-bond0

DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.1.100
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
BONDING_OPTS="mode=active-backup miimon=100"

/etc/sysconfig/network-scripts/ifcfg-ens33

DEVICE=ens33
NAME=ens33
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
ONBOOT=yes

/etc/sysconfig/network-scripts/ifcfg-ens34

DEVICE=ens34
NAME=ens34
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
ONBOOT=yes

// 구성이 완료된 이후에 모든 인터페이스를 재시작 한다.
nmcli connection reload
systemctl restart NetworkManager

```
### 2.9 본딩 상태 확인 

본딩 구성이 완료된 이후에는 별도의 시스템 파일을 이용하여 상태를 확인하거나 명령어를 이용하여 정상적으로 IP가 설정되어 동작하는지 확인할수 있다. 

```bash
cat /proc/net/bonding/bond0
...
ip addr show bond0
```
- 본딩 모드가 mode=4 (802.3ad)인 경우 스위치에서도 LACP를 지원하도록 구성해야 한다..
- miimon 옵션은 네트워크 링크 상태를 확인하는 주기를 밀리초 단위로 설정한다 (기본값: 100ms)
 
