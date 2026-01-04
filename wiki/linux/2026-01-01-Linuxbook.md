---
layout: post
title:  " 좌충우돌 리눅스 책쓰기 "
date:   2026-01-01 15:10:36 +0900
categories: [Linux, Book]
---

[0. 머리말 : 이책을 시작하기전](#0-클라우드-시대에-리눅스를-써야되) \
[1. 네트워킹을 시작해 보자](#1-네트워킹을-시작해-보자) \
[2. 리눅스 커널의 순서를 변경해 보자](#2-리눅스-커널-순서-변경)

-------

### 0. 클라우드 시대에 리눅스를 써야되?

준비중

-------

### 1. 네트워킹을 시작해 보자 

**1.1 Active 인터페이스 찾기**

```java
nmcli device status

DEVICE   TYPE      STATE      CONNECTION
ens33    ethernet  connected  Wired_connection_1
lo       loopback  unmanaged  --
```

**1.2 네트워크 연결 생성 (고정ip)**

```java
nmcli connection add type ethernet con-name my-ethernet ifname ens33 
ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
```
- **`type ethernet`** : 이더넷 연결 유형.
- **`con-name`** : 연결 이름 지정.
- **`ifname`** : 네트워크 인터페이스 이름(예: ens33).
- **`ipv4.addresses`** : IP 주소와 서브넷 마스크(예: 192.168.1.100/24).
- **`ipv4.gateway`** : 기본 게이트웨이.
- **`ipv4.dns`** : DNS 서버.
- **`ipv4.method manual`** : 고정 IP 설정(자동 할당은 auto).

**1.3 네트워크 연결 활성화**

```bash
nmcli connection up my-ethernet [지정된 연결이름]
```
**1.4 네트워크 연결 비활성화**

```bash
nmcli connection down my-ethernet [지정된 연결이름]
```
**1.5 네트워크 연결 DHCP**

```bash
nmcli connection modify my-ethernet ipv4.method auto
nmcli connection up my-ethernet
```

**1.6 네트워크 설정파일 편집** 

네트워크 설정은 **`/etc/sysconfig/network-scripts/ifcfg-<인터페이스 이름>`** 파일에서 관리된다. 

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33

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
**2. 네트워크 Bonding 구성하기**

**2.1 bonding 구성하기전**

사용 가능한 인터페이스 찾기. 

```bash
nmcli device status

DEVICE   TYPE      STATE      CONNECTION
ens33    ethernet  connected  Wired_connection_1
ens34    ethernet  connected  Wired_connection_1
lo       loopback  unmanaged  --
```
본딩을 구성하기 위한 ens33, ens34 인터페이스를 확인한다. 

**2.2 bonding 모드 이해**

본딩 모드 이해 본딩 모드는 네트워크 트래픽을 처리하는 방식에 따라 다릅니다.

```bash
- mode=0 (balance-rr) : 라운드 로빈 방식(트래픽 부하 분산).
- mode=1 (active-backup) : 활성-대기 방식(고가용성 제공).
- mode=2 (balance-xor) : XOR 방식(특정 알고리즘 기반 트래픽 분산).
- mode=4 (802.3ad) : LACP(Link Aggregation Control Protocol) 사용. / 스위치에서 지원가능해야 함.
- mode=5 (balance-tlb) : 송신 트래픽만 부하 분산.
- mode=6 (balance-alb) : 송신 및 수신 트래픽 부하 분산.
```
**2.3 nmcli를 이용한 bonding 구성**

```bash
nmcli connection add type bond con-name bond0 ifname bond0 mode active-backup
```
**주요 옵션** :
- **`type bond`** : 본딩 인터페이스 생성.
- **`con-name bond0`**: 본딩 연결 이름 지정.
- **`ifname bond0`** : 본딩 인터페이스 이름.
- **`mode active-backup`**: 본딩 모드 설정(예: active-backup).

**2.4 slave 인터페이스 추가**

bond0 인터페이스에 포함될 물리 네트워크 인터페이스를 슬레이브로 설정한다.

```bash
nmcli connection add type ethernet con-name ens33-slave ifname ens33 master bond0 [첫번째 인터페이스]

nmcli connection add type ethernet con-name ens34-slave ifname ens34 master bond0 [두번째 인터페이스]
```

**2.5 bond0 인터페이스에 ip추가**

```bash
nmcli connection modify bond0 ipv4.addresses 192.168.1.100/24 
ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
```
**2.6 bond0 인터페이스 dhcp 설정**

```bash
nmcli connection modify bond0 ipv4.method auto
```
**2.7 bond0 인터페이스 활성화**

bonding 인터페이스가 구성이 완료되면 모든 물리 인터페이스 까지 활성화 한다.

```bash
nmcli connection up bond0
nmcli connection up ens33-slave
nmcli connection up ens34-slave
```

**2.8 본딩 인터페이스 파일 생성**

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
**2.9 본딩 상태 확인**

본딩 구성이 완료된 이후에는 별도의 시스템 파일을 이용하여 상태를 확인하거나 명령어를 이용하여 정상적으로 IP가 설정되어 동작 확인

```bash
cat /proc/net/bonding/bond0
...
ip addr show bond0
```
- 본딩 모드가 mode=4 (802.3ad)인 경우 스위치에서도 LACP를 지원하도록 구성해야 한다..
- miimon 옵션은 네트워크 링크 상태를 확인하는 주기를 밀리초 단위로 설정한다 (기본값: 100ms)

-------

### 2. 리눅스 커널 순서 변경 

**1. 커널 순서 바꾸기**

* 리눅스 시스템에서 커널 업데이트 이후에 새롭게 설치된 커널 버젼에 대한 확인 및 순서 변경 
* 클라우드/온프렘 시스템 변경 작업에 있어서는 은근 손이 많이 가는 작업이기는 하다. 

**2. 부팅 가능한 커널 확인**

```bash 
sudo grubby --info=ALL | grep ^kernel
..
kernel=/boot/vmlinuz-5.14.0-427.13.1.el9_4.x86_64
kernel=/boot/vmlinuz-5.14.0-362.24.2.el9_3.x86_64
```

**3. Grub 설정에서 직접 추출**
* (1) BIOS 기반의 시스템에 설치된 커널 
* (2) UFI 기반의 시스템에 설치된 커널

```bash 
(1) awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
(2) awk -F\' '$1=="menuentry " {print $2}' /etc/grub2-efi.cfg
```

**4. 커널 Default 변경 및 확인**
* 커널에 대한 기본 설정 부터 grub.cfg 파일 재생성 절차 

```bash 
(1) sudo grub2-set-default 'Rocky Linux (5.14.0-427.13.1.el9_4.x86_64) 9.4 (Blue Onyx)'
(2) sudo grub2-editenv list
(3) sudo grub2-mkconfig -o /boot/grub2/grub.cfg
(4) sudo grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg
```
- 5.grubby 명령을 사용한 Default 커널 변경 및 확인
* grubby 명령을 이용한 기본 커널 설정 및 확인 방법

```bash 
(1) sudo grubby --set-default /boot/vmlinuz-5.14.0-427.13.1.el9_4.x86_64
(2) sudo grubby --default-kernel
```

**5. 커널 변경 자동화 스크립트**
* BIOS/UEFI 자동 감지 → 설치된 커널 목록 출력 → 번호 선택 → 기본 부팅 커널 변경 순서로 동작
* Rocky Linux 9(BLS 기반)에서는 grubby가 가장 안전하므로 우선 사용하고, 없으면 GRUB 메뉴 타이틀을 이용한 대안을 제공.
* 저장: /usr/local/sbin/set-default-kernel.sh (루트로 실행)

```bash 
#!/usr/bin/env bash
#
# set-default-kernel.sh
# Rocky Linux 9: Detect BIOS/UEFI -> list installed kernels -> set default kernel
# Requires: grubby (preferred). Falls back to grub2-set-default if grubby is missing.

set -euo pipefail

# --- Helpers ---
die() { echo "ERROR: $*" >&2; exit 1; }
need_root() {
  if [[ $EUID -ne 0 ]]; then
    echo "[i] Root 권한이 필요합니다. sudo로 재실행합니다..."
    exec sudo -E "$0" "$@"
  fi
}

# --- Ensure root ---
need_root "$@"

# --- Detect firmware ---
if [[ -d /sys/firmware/efi ]]; then
  FW="UEFI"
  GRUB_CFG_READ="/etc/grub2-efi.cfg"
  GRUB_CFG_WRITE="/boot/efi/EFI/rocky/grub.cfg"
else
  FW="BIOS"
  GRUB_CFG_READ="/etc/grub2.cfg"
  GRUB_CFG_WRITE="/boot/grub2/grub.cfg"
fi

echo "[i] Firmware : $FW"
echo "[i] GRUB cfg : read=$GRUB_CFG_READ, write=$GRUB_CFG_WRITE"

HAVE_GRUBBY=0
if command -v grubby >/dev/null 2>&1; then
  HAVE_GRUBBY=1
fi

CURRENT_RUNNING="$(uname -r || true)"

echo
echo "===== Installed kernels ====="
KERNELS=()        # e.g. /boot/vmlinuz-5.14.0-...
TITLES=()         # GRUB menu titles

if [[ $HAVE_GRUBBY -eq 1 ]]; then
  # Collect from grubby (BLS-aware)
  mapfile -t KERNELS < <(grubby --info=ALL | awk -F= '/^kernel=/{print $2}')
  mapfile -t TITLES  < <(grubby --info=ALL | awk -F= '/^title=/{print $2}')
  DEFAULT_KERNEL="$(grubby --default-kernel 2>/dev/null || true)"
  DEFAULT_TITLE="$(grubby --info="$DEFAULT_KERNEL" 2>/dev/null | awk -F= '/^title=/{print $2}')"
else
  echo "[!] grubby가 없습니다. 가능한 경우 설치를 권장합니다: dnf install -y grubby"
  # Fallback: parse GRUB menu entries (exclude rescue)
  mapfile -t TITLES < <(awk -F"'" '$1=="menuentry " && $2 !~ /rescue/ {print $2}' "$GRUB_CFG_READ" 2>/dev/null || true)
  # We won't have /boot/vmlinuz paths reliably; fill placeholders
  for _ in "${TITLES[@]}"; do KERNELS+=("N/A-without-grubby"); done
  DEFAULT_TITLE="$(/usr/bin/grub2-editenv list 2>/dev/null | awk -F= '$1=="saved_entry"{print $2}')"
  DEFAULT_KERNEL=""
fi

if [[ ${#TITLES[@]} -eq 0 ]]; then
  die "설치된 커널 목록을 찾지 못했습니다."
fi

for i in "${!TITLES[@]}"; do
  idx=$((i+1))
  mark=""
  if [[ -n "${DEFAULT_TITLE:-}" && "${TITLES[$i]}" == "$DEFAULT_TITLE" ]]; then
    mark="*default"
  fi
  # Try to derive version for nicer display
  ver="${KERNELS[$i]##*/vmlinuz-}"
  if [[ "$ver" == "${KERNELS[$i]}" || -z "$ver" ]]; then
    # attempt from title: "Rocky Linux (5.14.0-...) ..."
    ver="$(sed -n 's/.*(\(.*\)).*/\1/p' <<< "${TITLES[$i]}")"
  fi
  running_mark=""
  if [[ -n "$CURRENT_RUNNING" && "$ver" == "$CURRENT_RUNNING" ]]; then
    running_mark="(running)"
  fi
  printf "%2d) %-70s  %-28s %s\n" "$idx" "${TITLES[$i]}" "$ver" "$mark $running_mark"
done

echo
read -rp "-> 기본 부팅으로 설정할 번호를 입력하세요: " CHOICE
if ! [[ "$CHOICE" =~ ^[0-9]+$ ]] || (( CHOICE < 1 || CHOICE > ${#TITLES[@]} )); then
  die "잘못된 선택입니다."
fi

SEL_INDEX=$((CHOICE-1))
SEL_TITLE="${TITLES[$SEL_INDEX]}"
SEL_KERNEL="${KERNELS[$SEL_INDEX]}"

echo
echo "[i] 선택된 항목:"
echo "    Title : $SEL_TITLE"
echo "    Kernel: ${SEL_KERNEL}"

# --- Apply change ---
if [[ $HAVE_GRUBBY -eq 1 && -n "$SEL_KERNEL" && "$SEL_KERNEL" != "N/A-without-grubby" ]]; then
  echo "[i] grubby로 기본 커널 설정 중..."
  grubby --set-default "$SEL_KERNEL"
  NEW_DEFAULT="$(grubby --default-kernel || true)"
  echo "[i] 기본 커널 -> $NEW_DEFAULT"
else
  echo "[i] grubby 미사용 경로: GRUB saved_entry를 타이틀로 설정합니다."
  grub2-set-default "$SEL_TITLE"
  # 재생성(보수적): 일부 환경에서 필요치 않을 수 있으나 안전을 위해 반영
  if [[ -w "$GRUB_CFG_WRITE" ]]; then
    echo "[i] grub2-mkconfig 실행 중..."
    grub2-mkconfig -o "$GRUB_CFG_WRITE" >/dev/null
  else
    echo "[!] $GRUB_CFG_WRITE 쓰기가 불가하여 mkconfig를 건너뜁니다."
  fi
  echo "[i] 현재 saved_entry:"
  grub2-editenv list || true
fi

echo
echo "완료되었습니다. 재부팅 후 적용됩니다."
echo "현재 실행 중인 커널: $CURRENT_RUNNING"
```
* 목록에서 번호 입력 → 해당 커널이 기본 부팅 항목으로 설정.
* grubby가 있으면 BLS 엔트리를 안전하게 갱신하고 
* 없을 때는 grub2-set-default로 **GRUB 저장 엔트리(saved_entry)**를 변경
* 적용은 재부팅 후 반영.