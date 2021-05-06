---
layout: post
title:  "Pacemaker High CPU Load Detected Event 발생"
date:   2021-05-07 00:13:36 +0530
categories: Linux Pacemaker,Azure cloud
---

## 증상 

Pacemaker가 구성되어 있는 Linux 시스템에서 아래와 같이 Message 로그에 High CPU load detected 메세지가 지속적으로 발생하는 경우가 있습니다 

```javascript
Aug 13 01:30:16 crmd[482]:   notice: throttle_handle_load: High CPU load detected: 17.090000
Aug 13 05:30:16 crmd[482]:   notice: throttle_handle_load: High CPU load detected: 16.340000
Aug 13 09:30:16 crmd[482]:   notice: throttle_handle_load: High CPU load detected: 54.799999
Aug 13 09:30:46 crmd[482]:   notice: throttle_handle_load: High CPU load detected: 33.209999
Aug 13 09:31:16  crmd[482]:   notice: throttle_handle_load: High CPU load detected: 20.129999
Aug 13 10:30:16 crmd[482]:   notice: throttle_handle_load: High CPU load detected: 47.340000
```

## 원인 

해당 이벤트는 해당 클러스터의 높은 Load Average로 인해서 노드에서 
작업을 조절하고 있음을 나타내는 이벤트로 시스템의 Load가 높아져
throttle_mode가 high가될경우 Delay가 발생해 Timeout으로 인한 Fail Over가 발생할수 있습니다. 

``````C
throttle_get_total_job_limit(int l)
{
    /* Cluster-wide limit */
    GHashTableIter iter;
    int limit = l;
    int peers = crm_active_peers();
    struct throttle_record_s *r = NULL;

    g_hash_table_iter_init(&iter, throttle_records);

    while (g_hash_table_iter_next(&iter, NULL, (gpointer *) &r)) {
        switch(r->mode) {

            case throttle_extreme:
                if(limit == 0 || limit > peers/4) {
                    limit = QB_MAX(1, peers/4);
                }
                break;

            case throttle_high:
                if(limit == 0 || limit > peers/2) {
                    limit = QB_MAX(1, peers/2);
                }
                break;
            default:
                break;
        }
    }

``````````

pacemaker에서 throttle_mode (high,medium,low)에 대한 Load에 대한 계산방식은 아래와 같습니다. 

``````````C
static enum throttle_state_e
throttle_handle_load(float load, const char *desc, int cores)
{
    float normalize;
    float thresholds[4];

    if (cores == 1) {
        /* On a single core machine, a load of 1.0 is already too high */
        normalize = 0.6;

    } else {
        /* Normalize the load to be per-core */
        normalize = cores;
    }
    thresholds[0] = throttle_load_target * normalize * THROTTLE_FACTOR_LOW;
    thresholds[1] = throttle_load_target * normalize * THROTTLE_FACTOR_MEDIUM;
    thresholds[2] = throttle_load_target * normalize * THROTTLE_FACTOR_HIGH;
    thresholds[3] = load + 1.0; /* never extreme */

    return throttle_check_thresholds(load, desc, thresholds);
}
#endif

[참조코드]: pacemaker github https://github.com/ClusterLabs/pacemaker/blob/master/daemons/controld/controld_throttle.c
``````````

이전 코드에서는 throttle_load_target 값에 load-threshold를 100을 기준으로 나눴을때 default load를 산정했으나 변경된 code에서는 core당 산정해 놓은 Normalized 기준을 곱해서 Threadholds에 대한 FACTOR를 산정하는 방식으로 변경된것 같습니다. 

pacemaker 명령어를 이용해서 load-threshold 값을 확인할수 있는데 평균적으로 1분이상 80% 이상 유지하게 되면 시스템에 Load가 있는것으로 판단하고 THROTTLE_FACTOR_HIGH 값을 초과시 High CPU throttling에 걸리게 됨으로 시스템에 부하를 주는 근본적인 원인을 찾는것이 중요합니다. 

``````javascript
# pcs property show --all | grep threshold
load-threshod: 80%
```````