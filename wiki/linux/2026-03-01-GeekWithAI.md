---
layout: post
title:  " 리눅스 엔지니어의 인공지능 활용기 "
date:   2026-02-15 11:00:36 +0900
categories: [리눅스, AI, 문제해결하기]
---

- 세상은 인공지능 시대로 가는데 아직도 **까만 화면 (터미널)**이 좋은 엔지니어의 이야기
- SRE/DevOps를 위한 현업 중심 AI 프롬팅 
- 장애 대응 / 운영 자동화 / 문제 해결을 위한 이야기
- 성능 분석을 위한 복잡한 RAW 데이타에 눈 뒤집히는 엔지니어의 활용기 
- 뭔가 이것저것 다 해보고 안되는거 또 해보는 이야기 

----

### 1. 먹고 살려면 리눅스 해야되.!! 

- 시대가 변했어도 리눅스는 써야겠고 서점에서 사는 책들은 벽돌에 가깝다, 리눅스에 대한 러닝 커브를 극복하자니 너무 구닥다리 기술이다. 
- 시간을 투자해서 공부하려니 너무 애매하고 업무는 빠르게 진행되서 뭔가 빠르게 적응하고 반영할수 있는 방법이 필요하다. 
- 클라우드 / 온프렘 어쨌든 리눅스다, SRE / DevOps / 성능 엔지니어링 어쨌든 내가 다해야되

<details class="custom-details">
  <summary>1. 역할에 대한 페르소나 정의</summary>
  <div class="details-content" style="padding: 15px; line-height: 1.8;">
    • <span class="txt-bg-gray">트리거:</span> p99 폭증, 스루풋 급락, Migration 실패 지속.<br>
    • <span class="txt-bg-gray">조치:</span> 서비스 안정성을 위해 강도 높은 물리적 제어를 실행.<br>
        • <span class="txt-bg-gray">드레인(Drain):</span> 일부 모델이나 테넌트를 강제 축소하거나 요청을 차단.<br>
        • <span class="txt-bg-gray">격리:</span> 해당 노드를 'Degraded'로 마킹하여 스케줄러가 새 워크로드를 배치하지 못하게 함.<br>
    • <span class="txt-bg-gray">재시작:</span> 누적된 파편화(Fragmentation) 해결을 위해 프로세스 롤링 재배치를 수행.
  </div>
</details>
