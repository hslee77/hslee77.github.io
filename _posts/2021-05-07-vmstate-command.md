---
layout: post
title:  "vmstat 명령어를 무시하지 마세요!"
date:   2021-05-07 10:45:20 +0530
categories: Linux performance,cloud
---
 
리눅스 시스템에 대한 성능을 측청하는데 있어서, 별도의 sms 툴을 사용하는 것도 좋은 방법이되기도 하겠지만 Linux에서 제공해주는 명령어들만 가지고도 충분히 Linux Performance에 대한 이해를 높일수 있습니다.

![screenshot](/img/linuxtech/linuxperformance.png)

[출처:Brendan Gregg][linuxperformance]

전체 하드웨어의 동작을 이해하고, CPU부터 Memory 그리고 Disk I/O에 대한 항목들을 이해한다면, 생각보다 상세한 시스템 Linux 시스템 성능에 대한 자세한 Mesurement가 가능하리라 생각됩니다, 특히 툴이 제공하는 모니터링의 Obseravaility의 scope이 어디까지인지 이해하고 있다면 단순하게 보여졌던 명령어에 또다른 면을 느끼게 될것입니다.

## vmstat는 뭐하는 녀석인가? 

linux에서 vmstat 명령어는 프로세스,CPU의 동작상태,메모리,
페이징 블록에 대한 모니터링 정보와 함께 I/O의 트랩까지 상태정보를 통해서 Linux 자체에서 발생하는 **병목(Bottleneck)증상**의 분석을 가능하게 해줍니다.

vmstat의 명령어가 보여준 결과는 Linux가 재부팅이 되기전까지 누적된 데이타를 보여주지만 CPU와 Memory 상태정보의 경우 즉각적으로 수집된 데이타를 보여주게 됩니다

## vmstat basic Field description

```javascript
Procs
       r: The number of runnable processes (running or waiting for run time).
       b: The number of processes blocked waiting for I/O to complete.

   Memory
       These are affected by the --unit option.
       swpd: the amount of swap memory used.
       free: the amount of idle memory.
       buff: the amount of memory used as buffers.
       cache: the amount of memory used as cache.
       inact: the amount of inactive memory. (-a option)
       active: the amount of active memory. (-a option)

   Swap
       These are affected by the --unit option.
       si: Amount of memory swapped in from disk (/s).
       so: Amount of memory swapped to disk (/s).

   IO
       bi: Blocks received from a block device (blocks/s).
       bo: Blocks sent to a block device (blocks/s).

   System
       in: The number of interrupts per second, including the clock.
       cs: The number of context switches per second.

   CPU
       These are percentages of total CPU time.
       us: Time spent running non-kernel code.  (user time, including nice time)
       sy: Time spent running kernel code.  (system time)
       id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
       wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
       st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.
```

[Mang Page for vmstat][vmstat] 매뉴얼 페이지

[Bredangregg vmstat][Brendangregg] 강의 영상

특히 **st** 항목의 경우 가상머신이 vCPU를 동작시키기 위해서 실제 물리 CPU를 대기하는 시간으로 가상화 환경의 경우 리소스를 공유해서 사용하기 때문에 물리 CPU를 대기하는데 이 상태를 백분율로 나타는 값이 st 항목이 됩니다. st값이 증가한다는 **물리CPU 리소스에서 해당vm으로 리소스 할당이 늦어진다는** 의미가 되기도 합니다. 

## Steal이 높아지면 나타나는 증상들? 
- vmstat로 인해서 st의 Usage가 높아졌다면 CPU에서 프로세스를 처리해야 되는 Queue에도 영향을 미칠수 있음으로 **runq**의 비율이 높아지는가 확인함과 동시에 I/O에서 ***Blocking (b)되는 프로세스의 비율**이 증가하는지도 확인해볼 필요가 있습니다.

- SMS 모니터링을 사용하고 있다면 전체적인 시스템의throttling이 높아지게 되고 서비스에 영향을 미치게 됩니다. 


## Steal을 낮추기 위한 방법은?
- 리소스 자원에 대한 제한 설정 확인,특정VM이 CPU 사용률이 높을경우 이로인한 영향을 받을수 있습니다.
- 물리 리소스가 충분한 Node로 VM 이동을 고려해볼수 있습니다.

- 부하가 높은 VM을 다른 Node로 분산시켜 CPU의 리소스를 평준화 시키는 시도도 의미가 있습니다.
- 물리적인 Node에 대한 리소스에 대한 scaleup 확장을 고려해야합니다.

## 퍼블릭 클라우드에서 조치사항 

- 퍼블릭 클라우드 상에서는 사용자가 직접 Resource 이슈에 대해서 직접적으로 핸들링 할수 없습니다.

- VM의 Size 보다 Throttling이 높게 나타나게 되면 VMSS를 확장하거나 VM size를 더 큰것으로 업그레이는 하는 방안을 고려할수 있습니다. 

- 성능과 관련된 Azure 문서의 Article을 참고하세요 

```
[AzureLinuxPerformance][azurelinuxperformance]
[AzureWindowPerfornace][azurewinperformance]
```





[linuxperformance]: http://www.brendangregg.com/Perf/linux_observability_tools.png

[vmstat]: https://man7.org/linux/man-pages/man8/vmstat.8.html

[azurelinuxperformance]:https://docs.microsoft.com/en-us/troubleshoot/azure/virtual-machines/troubleshoot-performance-virtual-machine-linux-windows

[azurewinperformance]: https://docs.microsoft.com/en-us/troubleshoot/azure/virtual-machines/troubleshoot-high-cpu-issues-azure-windows-vm

[Brendangregg]:https://www.youtube.com/watch?v=k9eX1jQR1hA