---
layout: post
title:  " AI데이타센터 성능 이야기 "
date:   2026-02-15 11:00:36 +0900
categories: [리눅스, AI데이타센터, 성능]
---
[1. 트렌드의 변화 : 시장은 지금 어떻게 변화하고 있나?](#1-트렌트의-변화) <br>
[2. 대용량 데이타의 고속 전송 : HBM 메모리는 어떻게 동작하나?](#2-hbm-메모리는-어떻게-동작하나)


### 1. 트렌트의 변화

- **2026.02.07 :** Brendan Gregg가 인텔을 떠나서 OpenAI로 합류 했다 !AI 데이터센터 성능 문제를 **기술적 챌린지를 넘어 환경·비용·사회 문제로 확장된 도전**으로 인식했고, 이를 직접 해결하기 위해 OpenAI에 합류했다는 철학적·전문적 이유를 그의 블로그를 통해서 확인이 가능하다.이제 인공지능 시대로 넘어가면서 고성능 데이타 센터에 대한 성능과 비용 그리고 기반 인프라에 요구되는 시설까지 (전원, 쿨링) 이제는 현실화로 이어져야 할 시기가 다가온듯 하다. [참고:Brendan Gregg :Why I joined OpenAI](https://www.brendangregg.com/blog/2026-02-07/why-i-joined-openai.html?fbclid=IwY2xjawP-tSVleHRuA2FlbQIxMQBzcnRjBmFwcF9pZBAyMjIwMzkxNzg4MjAwODkyAAEeTzBai6GBFHPv6l1FU_MIG0rJTdBRbCIvKRBj625T-DWm9pnmubjbQ-rnlo0_aem_18ye9NFVcv4Nsmsq_DD5sg)

- **2026.02.11 :**DDR 메모리 가격이 레디컬하게 증가하기 시작했다 삼성이나 하이닉스와 같은 메모리 제조사들은 AI 데이타 센터의 확장으로 HBM 메모리 생산으로 집중하기 시작하면서 최근의 메모리 가격 상승은 단순한 공급망 쇼크라기보다, **AI 인프라 확대와 산업 구조 변화가 겹친 구조적 부족 상황이라는 평가가 지배적이다** 개인적인 관점이기는 하지만 이제 리소스 관점에서의 엔지니어링 기법이 다시 필요한 시점이 될듯하다. 유닉스에서 리눅스로 전환될때 어플리케이션의 프레임 워크는 Posix 표준에 맞춰져서 개발해야 하고 리소스에 대한 활용에 대한 관점을 바꿔야 했다. 작게는 리눅스지만 좀더 크게는 시스템 성능 엔지니어링 관점의 리소스 활용에 대한 전략이 무엇보다 필요한 시대가 되어간다.<br>
[참고자료 : 2024–present global memory supply shortage](https://en.wikipedia.org/wiki/2024%E2%80%93present_global_memory_supply_shortage?utm_source=chatgpt.com&fbclid=IwY2xjawP-tdZleHRuA2FlbQIxMQBzcnRjBmFwcF9pZBAyMjIwMzkxNzg4MjAwODkyAAEe1g-K-3fYkPNpPnnWQ_8n2jtx-7JdBCJX7qgLFExVbTWZmUBSlJB5l3sBT0M_aem_s5wdudyfXpfSpyuoQf7-cQ#)
<img src="/img/linuxtech/DDR5-5200_2x16GB_monthly_price_trend_en.png" alt="screenshot" align=left width="800"/>
<div style="clear:both;"></div>

### 2. HBM 메모리는 어떻게 동작하나?

- **2026.02.16 :** HBM은 “연산 유닛(GPU/AI Accelerator)이 굶지 않도록 초고속으로 데이터를 공급하는 메모리”
- HBM은 메모리의 클럭 보다는 고대역폭을(1024bit) **(낮은 클럭 + 매우 넓은 버스 = 고대역폭 + 낮은 전력)** 으로 데이타를 GPU에 공급 
- HBM은 Cache의 역할이 아니며 **연산 전용 고속 주 메모리**
- 실제 동작 흐름 (AI 워크로드 기준)  **Transformer 모델 추론 --> GPU 코어(SM)가 연산 요청 --> 메모리 컨트롤러가 HBM에 병렬 요청 --> 수천 비트 폭으로 텐서 데이터 전달 --> 연산 유닛은 대기 없이 연산 지속**
- OS/런타임은 **어떤 데이터를 HBM에 둘 것인가?** 를 결정하는 쪽으로 진화 중
- 리눅스 NUMA와 메모리 tiering에서 HBM은 “가장 빠른 NUMA 노드로 hot data를 끌어올리는 성능 가속 레이어”로 사용된다.

----------
<img src="/img/aitech/20260216_HBM.png" alt="screenshot" align=left width="800"/>
<div style="clear:both;"></div>