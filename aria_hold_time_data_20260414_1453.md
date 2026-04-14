# ARIA v2 — Hold Time Paralysis Data Collection

*Generated: 2026-04-14T14:53:53+07:00*

Column names di prompt diadjust ke actual schema (entry_idr→entry_value_idr, sell_idr→sell_value_idr).

---

## PART 1: Position Detail (v2_positions)

```
id  agent  coin  entry_price   high_price    sell_price  entry_idr     sell_idr          pnl_idr            pnl_pct              status      opened_at                         closed_at                         hold_min  high_pct  realized_pct
--  -----  ----  ------------  ------------  ----------  ------------  ----------------  -----------------  -------------------  ----------  --------------------------------  --------------------------------  --------  --------  ------------
11  A2     ARB   1915.0        1972.0        0.0         23394.49175   0.0               0.0                0.0                  STUCK_DUST  2026-04-10T05:54:00.872158+00:00  2026-04-10T22:22:04.446654+00:00  988.1     2.977     -100.0      
12  A1     DOGE  1591.5        1596.5        1592.0      53473.124     53489.9235991203  16.7995991203267   0.0314169022934339   SOLD        2026-04-10T05:54:00.883753+00:00  2026-04-10T12:54:28.495453+00:00  420.5     0.314     0.031       
13  A1     ETH   37690944.5    38123597.5    38015012.0  23103.764     23302.410628239   198.646628239046   0.859802014247746    SOLD        2026-04-10T05:54:06.190961+00:00  2026-04-10T11:56:53.475809+00:00  362.8     1.148     0.86        
14  A1     BTC   1237634625.5  1255593000.0  0.0         23126.512     0.0               0.0                0.0                  STUCK_DUST  2026-04-10T11:57:03.992774+00:00  2026-04-10T16:18:38.937304+00:00  261.6     1.451     -100.0      
15  A1     DOGE  1592.0        1597.5        1593.5      39086.904     39123.7321130653  36.8281130653266   0.0942211055276382   SOLD        2026-04-10T12:54:38.955006+00:00  2026-04-10T13:27:09.014852+00:00  32.5      0.345     0.094       
16  A1     ETH   38052016.0    38610720.5    38506754.0  39012.948     39479.1695517733  466.221551773341   1.19504312202539     SOLD        2026-04-10T13:27:14.336150+00:00  2026-04-10T14:27:16.591730+00:00  60.0      1.468     1.195       
17  A3     SUI   16065.5       16480.5       15569.5     88382.76615   0.0               -2728.69515485979  -0.0308736111543369  SOLD        2026-04-10T13:54:43.498583+00:00  2026-04-12 12:36:30               2801.8    2.583     -3.087      
18  A2     SOL   1444757.5     1466868.5     0.0         147304.61025  0.0               0.0                0.0                  STUCK_DUST  2026-04-10T13:54:43.627269+00:00  2026-04-10T20:41:13.629261+00:00  406.5     1.53      -100.0      
19  A1     DOGE  1609.5        1632.0        0.0         257782.688    0.0               0.0                0.0                  STUCK_DUST  2026-04-10T14:27:22.001571+00:00  2026-04-10T19:39:17.298704+00:00  311.9     1.398     -100.0      
```


---

## PART 2: Signal Log (v2_signals)

```
id   from_age  signal_type      coin    detail                                                        created_at                 handled  b2_response                               handled_at               
---  --------  ---------------  ------  ------------------------------------------------------------  -------------------------  -------  ----------------------------------------  -------------------------
2    A2        DISTRESS         ARB     {"entry_price": 1915.0, "current_price": 1857.5, "pool": "ID  2026-04-10T06:35:02.84841  1        {"decision": "HOLD", "reason": "recent t  2026-04-10T06:35:05.35988
                                        R", "pnl_pct": -3.0026109660574414}                           1+00:00                             rend +2.4% (recovering); price -2.2% bel  8+00:00                  
                                                                                                                                          ow avg; moderate loss -3.0%", "confidenc                           
                                                                                                                                          e": 0.5, "pnl_pct": -3.0, "recent_trend"                           
                                                                                                                                          : 2.36, "price_vs_avg": -2.19, "data_poi                           
                                                                                                                                          nts": 7}                                                           

3    A2        DISTRESS         ARB     {"entry_price": 1915.0, "current_price": 1854.0, "pool": "ID  2026-04-10T06:37:04.13803  1        {"decision": "HOLD", "reason": "recent t  2026-04-10T06:37:05.37695
                                        R", "pnl_pct": -3.185378590078329}                            9+00:00                             rend +2.4% (recovering); price -2.4% bel  2+00:00                  
                                                                                                                                          ow avg; moderate loss -3.2%", "confidenc                           
                                                                                                                                          e": 0.5, "pnl_pct": -3.19, "recent_trend                           
                                                                                                                                          ": 2.36, "price_vs_avg": -2.37, "data_po                           
                                                                                                                                          ints": 7}                                                          

4    A2        DISTRESS         ARB     {"entry_price": 1915.0, "current_price": 1854.5, "pool": "ID  2026-04-10T06:39:05.44683  1        {"decision": "HOLD", "reason": "recent t  2026-04-10T06:39:08.31347
                                        R", "pnl_pct": -3.1592689295039165}                           6+00:00                             rend +2.4% (recovering); price -2.3% bel  2+00:00                  
                                                                                                                                          ow avg; moderate loss -3.2%", "confidenc                           
                                                                                                                                          e": 0.5, "pnl_pct": -3.16, "recent_trend                           
                                                                                                                                          ": 2.36, "price_vs_avg": -2.34, "data_po                           
                                                                                                                                          ints": 7}                                                          

5    A2        DISTRESS         ARB     {"entry_price": 1915.0, "current_price": 1856.0, "pool": "ID  2026-04-10T06:41:43.14735  1        {"decision": "HOLD", "reason": "recent t  2026-04-10T06:41:48.33713
                                        R", "pnl_pct": -3.0809399477806787}                           8+00:00                             rend +2.4% (recovering); price -2.3% bel  6+00:00                  
                                                                                                                                          ow avg; moderate loss -3.1%", "confidenc                           
                                                                                                                                          e": 0.5, "pnl_pct": -3.08, "recent_trend                           
                                                                                                                                          ": 2.36, "price_vs_avg": -2.26, "data_po                           
                                                                                                                                          ints": 7}                                                          

6    A2        DISTRESS         ARB     {"entry_price": 1915.0, "current_price": 1855.5, "pool": "ID  2026-04-10T06:43:44.45187  1        {"decision": "HOLD", "reason": "recent t  2026-04-10T06:43:48.36150
                                        R", "pnl_pct": -3.107049608355091}                            2+00:00                             rend +2.4% (recovering); price -2.3% bel  4+00:00                  
                                                                                                                                          ow avg; moderate loss -3.1%", "confidenc                           
                                                                                                                                          e": 0.5, "pnl_pct": -3.11, "recent_trend                           
                                                                                                                                          ": 2.36, "price_vs_avg": -2.29, "data_po                           
                                                                                                                                          ints": 7}                                                          

7    A1        SOLD             ETH     {"pnl_pct": 0.86, "pnl_idr": 199.0, "reason": "Trail stop: d  2026-04-10T11:56:53.48614  1        {"action": "replacement_assigned"}        2026-04-10T11:57:00.54032
                                        ropped 0.29% from high +1.15%"}                               9+00:00                                                                       3+00:00                  

8    A1        SOLD             DOGE    {"pnl_pct": 0.03, "pnl_idr": 17.0, "reason": "Trail stop: dr  2026-04-10T12:54:28.50429  1        {"action": "replacement_assigned"}        2026-04-10T13:27:09.12410
                                        opped 0.28% from high +0.31%"}                                2+00:00                                                                       8+00:00                  

9    A1        SOLD             DOGE    {"pnl_pct": 0.09, "pnl_idr": 37.0, "reason": "Trail stop: dr  2026-04-10T13:27:09.02230  1        {"action": "replacement_assigned"}        2026-04-10T13:27:09.13371
                                        opped 0.25% from high +0.35%"}                                1+00:00                                                                       2+00:00                  

10   A1        SOLD             ETH     {"pnl_pct": 1.2, "pnl_idr": 466.0, "reason": "Trail stop: dr  2026-04-10T14:27:16.60107  1        {"action":"manual_cleanup_pre_step5"}     2026-04-10 15:51:57      
                                        opped 0.27% from high +1.47%"}                                1+00:00                                                                                                

11   A1        DESYNC_DETECTED  BTC     {"v2_qty": 1.868605768092257e-05, "exchange_qty": 1.892e-05,  2026-04-10T15:54:49.22933  1        {"action": "logged"}                      2026-04-10T15:54:57.28931
                                         "delta_pct": 1.24, "root_cause_hint": "PARTIAL_FILL"}        2+00:00                                                                       9+00:00                  

12   A1        SOLD             BTC     {"pnl_pct": 0.81, "pnl_idr": 0, "reason": "STUCK_DUST: Trail  2026-04-10T16:18:38.94521  1        {"action": "replacement_assigned"}        2026-04-10T16:18:39.02602
                                         stop: dropped 0.64% from high +1.45%"}                       6+00:00                                                                       3+00:00                  

13   A1        DESYNC_DETECTED  DOGE    {"v2_qty": 160.163210935073, "exchange_qty": 0.940278, "delt  2026-04-10T19:39:17.29528  1        {"action": "logged"}                      2026-04-10T19:39:23.30009
                                        a_pct": 99.41, "root_cause_hint": "GHOST_POSITION"}           5+00:00                                                                       2+00:00                  

14   A1        SOLD             DOGE    {"pnl_pct": 0.99, "pnl_idr": 0, "reason": "STUCK_DUST: Trail  2026-04-10T19:39:17.30479  1        {"action": "replacement_assigned"}        2026-04-10T19:39:23.30773
                                         stop: dropped 0.40% from high +1.40%"}                       2+00:00                                                                       9+00:00                  

15   A2        DESYNC_DETECTED  SOL     {"v2_qty": 0.10195801734893226, "exchange_qty": 8.556e-05, "  2026-04-10T20:41:13.62582  1        {"action": "logged"}                      2026-04-10T20:41:20.78299
                                        delta_pct": 99.92, "root_cause_hint": "GHOST_POSITION"}       3+00:00                                                                       5+00:00                  

16   A2        SOLD             SOL     {"pnl_pct": 0.88, "pnl_idr": 0, "reason": "STUCK_DUST: Trail  2026-04-10T20:41:13.63599  1        {"action": "replacement_exhausted", "rea  2026-04-10T20:41:40.79652
                                         stop: dropped 0.65% from high +1.53%"}                       0+00:00                             son": "no slot after 3 attempts"}         9+00:00                  

17   A2        DESYNC_DETECTED  ARB     {"v2_qty": 12.216444778067883, "exchange_qty": 0.0183312, "d  2026-04-10T22:21:52.26768  1        {"action": "logged"}                      2026-04-10T22:21:58.55215
                                        elta_pct": 99.85, "root_cause_hint": "GHOST_POSITION"}        5+00:00                                                                       2+00:00                  

18   A2        SOLD             ARB     {"pnl_pct": 2.38, "pnl_idr": 0, "reason": "STUCK_DUST: Trail  2026-04-10T22:22:04.45308  1        {"action": "replacement_exhausted", "rea  2026-04-10T22:22:28.56631
                                         stop: dropped 0.60% from high +2.98%"}                       2+00:00                             son": "no slot after 3 attempts"}         3+00:00                  
```

### Signal Summary

```
from_agent  signal_type      count
----------  ---------------  -----
A1          DESYNC_DETECTED  2    
A1          SOLD             6    
A2          DESYNC_DETECTED  2    
A2          DISTRESS         5    
A2          SOLD             2    
```


---

## PART 3: Order Attempts (v2_order_attempts)

**Schema note:** no `coin` or `reason` column; fields adjusted to actual:
- `symbol` (pair form), `attempted_price`/`qty`, `filled_qty`/`price`, `fees_paid`, `attempted_at`/`completed_at`

```
id   agen  symbol           side  status      attempted_  attempted_  filled_pri  filled_qty  fees_paid   attempted_at            completed_at          
---  ----  ---------------  ----  ----------  ----------  ----------  ----------  ----------  ----------  ----------------------  ----------------------
1    A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:08.35  2026-04-10T20:40:08.35
                                                                                                          9224+00:00              9261+00:00            

2    A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:08.36  2026-04-10T20:40:08.36
                                  ARKET                                                                   3264+00:00              3273+00:00            

3    A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:11.90  2026-04-10T20:40:11.90
                                                                                                          6282+00:00              6301+00:00            

4    A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:11.91  2026-04-10T20:40:11.91
                                  ARKET                                                                   0679+00:00              0693+00:00            

5    A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:15.41  2026-04-10T20:40:15.41
                                                                                                          9395+00:00              9412+00:00            

6    A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:15.42  2026-04-10T20:40:15.42
                                  ARKET                                                                   3042+00:00              3050+00:00            

7    A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:19.27  2026-04-10T20:40:19.27
                                                                                                          4046+00:00              4061+00:00            

8    A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:19.27  2026-04-10T20:40:19.27
                                  ARKET                                                                   7899+00:00              7911+00:00            

9    A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:22.78  2026-04-10T20:40:22.78
                                                                                                          1056+00:00              1071+00:00            

10   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:22.78  2026-04-10T20:40:22.78
                                  ARKET                                                                   4429+00:00              4437+00:00            

11   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:26.30  2026-04-10T20:40:26.30
                                                                                                          4273+00:00              4289+00:00            

12   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:26.30  2026-04-10T20:40:26.30
                                  ARKET                                                                   8479+00:00              8502+00:00            

13   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:30.16  2026-04-10T20:40:30.16
                                                                                                          2860+00:00              2876+00:00            

14   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:30.16  2026-04-10T20:40:30.16
                                  ARKET                                                                   6173+00:00              6180+00:00            

15   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:33.66  2026-04-10T20:40:33.66
                                                                                                          8921+00:00              8937+00:00            

16   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:33.67  2026-04-10T20:40:33.67
                                  ARKET                                                                   2875+00:00              2883+00:00            

17   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:37.17  2026-04-10T20:40:37.17
                                                                                                          9861+00:00              9896+00:00            

18   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:37.18  2026-04-10T20:40:37.18
                                  ARKET                                                                   2830+00:00              2836+00:00            

19   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:41.03  2026-04-10T20:40:41.03
                                                                                                          5727+00:00              5742+00:00            

20   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:41.04  2026-04-10T20:40:41.04
                                  ARKET                                                                   1140+00:00              1156+00:00            

21   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:44.57  2026-04-10T20:40:44.57
                                                                                                          3419+00:00              3434+00:00            

22   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:44.57  2026-04-10T20:40:44.57
                                  ARKET                                                                   7404+00:00              7419+00:00            

23   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:48.08  2026-04-10T20:40:48.08
                                                                                                          9362+00:00              9377+00:00            

24   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:48.09  2026-04-10T20:40:48.09
                                  ARKET                                                                   3690+00:00              3718+00:00            

25   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:51.99  2026-04-10T20:40:51.99
                                                                                                          5439+00:00              5455+00:00            

26   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:51.99  2026-04-10T20:40:51.99
                                  ARKET                                                                   8666+00:00              8673+00:00            

27   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:55.50  2026-04-10T20:40:55.50
                                                                                                          1971+00:00              1987+00:00            

28   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:55.50  2026-04-10T20:40:55.50
                                  ARKET                                                                   5958+00:00              5973+00:00            

29   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:40:59.00  2026-04-10T20:40:59.00
                                                                                                          5548+00:00              5566+00:00            

30   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:40:59.00  2026-04-10T20:40:59.00
                                  ARKET                                                                   9815+00:00              9827+00:00            

31   A2    SOL_IDR          SELL  FAILED      1455380.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:41:02.90  2026-04-10T20:41:02.90
                                                                                                          1687+00:00              1703+00:00            

32   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:41:02.90  2026-04-10T20:41:02.90
                                  ARKET                                                                   5946+00:00              5955+00:00            

33   A2    SOL_IDR          SELL  FAILED      1455684.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:41:06.42  2026-04-10T20:41:06.42
                                                                                                          1413+00:00              1428+00:00            

34   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:41:06.42  2026-04-10T20:41:06.42
                                  ARKET                                                                   5509+00:00              5517+00:00            

35   A2    SOL_IDR          SELL  FAILED      1455684.0   0.10168556  0.0         0.0         0.0         2026-04-10T20:41:10.07  2026-04-10T20:41:10.07
                                                                                                          2399+00:00              2416+00:00            

36   A2    SOL_IDR          SELL  FALLBACK_M  0.0         0.10168556  0.0         0.0         0.0         2026-04-10T20:41:10.07  2026-04-10T20:41:10.07
                                  ARKET                                                                   6537+00:00              6545+00:00            

37   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:20:49.38  2026-04-10T22:20:49.38
                                                          780679                                          1334+00:00              1357+00:00            

38   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:20:49.38  2026-04-10T22:20:49.38
                                  ARKET                   780679                                          5969+00:00              6182+00:00            

39   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:20:52.84  2026-04-10T22:20:52.84
                                                          780679                                          0806+00:00              0823+00:00            

40   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:20:52.84  2026-04-10T22:20:52.84
                                  ARKET                   780679                                          3887+00:00              3894+00:00            

41   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:20:56.33  2026-04-10T22:20:56.33
                                                          780679                                          8614+00:00              8630+00:00            

42   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:20:56.34  2026-04-10T22:20:56.34
                                  ARKET                   780679                                          2413+00:00              2419+00:00            

43   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:00.05  2026-04-10T22:21:00.05
                                                          780679                                          2874+00:00              2902+00:00            

44   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:00.05  2026-04-10T22:21:00.05
                                  ARKET                   780679                                          6738+00:00              6745+00:00            

45   A2    ARB_IDR          SELL  FAILED      1954.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:03.51  2026-04-10T22:21:03.51
                                                          780679                                          3914+00:00              4245+00:00            

46   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:03.51  2026-04-10T22:21:03.51
                                  ARKET                   780679                                          7861+00:00              7867+00:00            

47   A2    ARB_IDR          SELL  FAILED      1954.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:06.94  2026-04-10T22:21:06.94
                                                          780679                                          8599+00:00              8618+00:00            

48   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:06.95  2026-04-10T22:21:06.95
                                  ARKET                   780679                                          2390+00:00              2643+00:00            

49   A2    ARB_IDR          SELL  FAILED      1955.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:10.49  2026-04-10T22:21:10.49
                                                          780679                                          8321+00:00              8337+00:00            

50   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:10.50  2026-04-10T22:21:10.50
                                  ARKET                   780679                                          2576+00:00              2586+00:00            

51   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:13.89  2026-04-10T22:21:13.89
                                                          780679                                          6386+00:00              6426+00:00            

52   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:13.90  2026-04-10T22:21:13.90
                                  ARKET                   780679                                          0093+00:00              0100+00:00            

53   A2    ARB_IDR          SELL  FAILED      1955.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:17.30  2026-04-10T22:21:17.30
                                                          780679                                          2445+00:00              2460+00:00            

54   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:17.30  2026-04-10T22:21:17.30
                                  ARKET                   780679                                          5839+00:00              5859+00:00            

55   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:20.85  2026-04-10T22:21:20.85
                                                          780679                                          8876+00:00              8891+00:00            

56   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:20.86  2026-04-10T22:21:20.86
                                  ARKET                   780679                                          2563+00:00              2570+00:00            

57   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:24.26  2026-04-10T22:21:24.26
                                                          780679                                          3882+00:00              3898+00:00            

58   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:24.26  2026-04-10T22:21:24.26
                                  ARKET                   780679                                          7545+00:00              7571+00:00            

59   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:27.66  2026-04-10T22:21:27.66
                                                          780679                                          9052+00:00              9070+00:00            

60   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:27.67  2026-04-10T22:21:27.67
                                  ARKET                   780679                                          3447+00:00              3456+00:00            

61   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:31.22  2026-04-10T22:21:31.22
                                                          780679                                          1976+00:00              1993+00:00            

62   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:31.22  2026-04-10T22:21:31.22
                                  ARKET                   780679                                          6198+00:00              6207+00:00            

63   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:34.79  2026-04-10T22:21:34.79
                                                          780679                                          8521+00:00              8537+00:00            

64   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:34.80  2026-04-10T22:21:34.80
                                  ARKET                   780679                                          1651+00:00              1660+00:00            

65   A2    ARB_IDR          SELL  FAILED      1956.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:38.20  2026-04-10T22:21:38.20
                                                          780679                                          4724+00:00              5221+00:00            

66   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:38.20  2026-04-10T22:21:38.20
                                  ARKET                   780679                                          8433+00:00              8439+00:00            

67   A2    ARB_IDR          SELL  FAILED      1958.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:42.05  2026-04-10T22:21:42.05
                                                          780679                                          1464+00:00              1482+00:00            

68   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:42.05  2026-04-10T22:21:42.05
                                  ARKET                   780679                                          6470+00:00              6479+00:00            

69   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:45.51  2026-04-10T22:21:45.51
                                                          780679                                          3885+00:00              3901+00:00            

70   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:45.51  2026-04-10T22:21:45.51
                                  ARKET                   780679                                          8207+00:00              8215+00:00            

71   A2    ARB_IDR          SELL  FAILED      1959.0      12.2164447  0.0         0.0         0.0         2026-04-10T22:21:48.97  2026-04-10T22:21:48.97
                                                          780679                                          0211+00:00              0228+00:00            

72   A2    ARB_IDR          SELL  FALLBACK_M  0.0         12.2164447  0.0         0.0         0.0         2026-04-10T22:21:48.97  2026-04-10T22:21:48.97
                                  ARKET                   780679                                          4499+00:00              4511+00:00            
```

### Order Attempts Summary

```
agent  side  status           n 
-----  ----  ---------------  --
A2     SELL  FAILED           36
A2     SELL  FALLBACK_MARKET  36
```


---

## PART 4: Exit Trigger Logic (aria_v2_runner.py)

### Grep — exit-related symbols

```
111:        "trail_activate_pct": 1.2,   # activate trail after +1.2% (fee-safe: net +0.35% after 0.3% RT fees + 0.15% spread)
112:        "trail_drop_pct":     0.4,   # sell if drops 0.4% from high (was 0.25%)
113:        "max_loss_pct":       1.5,   # distress signal at -1.5% (unchanged)
114:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 120s)
121:        "trail_activate_pct": 1.5,   # activate trail after +1.5% (was +1.0%)
122:        "trail_drop_pct":     0.6,   # sell if drops 0.6% from high (was 0.5%)
123:        "max_loss_pct":       3.0,   # distress signal at -3.0% (unchanged)
124:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 60s)
131:        "trail_activate_pct": 3.0,   # activate trail after +3.0% (was +2.0%)
132:        "trail_drop_pct":     1.2,   # sell if drops 1.2% from high (was 1.0%)
133:        "max_loss_pct":       5.0,   # distress signal at -5.0% (unchanged)
134:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 30s)
161:    max_loss_override: float = 0.0   # per-position override; 0 = use profile default
346:    # Also fetch assignment reasons for max_loss override detection
371:        # Restore RISKY-WL max_loss override from assignment reason
374:            pos.max_loss_override = 2.0
377:        override_tag = " [max_loss=-2% RISKY-WL]" if pos.max_loss_override else ""
581:    log.info(f"  Trail: activate +{profile['trail_activate_pct']}%, "
582:             f"drop {profile['trail_drop_pct']}%")
583:    log.info(f"  Max loss: {profile['max_loss_pct']}% | "
718:                            pos.max_loss_override = 2.0  # tighter -2% for WL dip buys
720:                                     f"max_loss tightened to -2%")
757:                if (pos.pnl_pct >= profile["trail_activate_pct"]
766:                    if (drop >= profile["trail_drop_pct"]
767:                            and hold_sec >= profile["min_hold_sec"]):
780:                # Use per-position max_loss_override if set (e.g., RISKY-WL entries)
781:                eff_max_loss = (pos.max_loss_override
782:                                if pos.max_loss_override > 0
783:                                else profile["max_loss_pct"])
784:                if (pos.pnl_pct <= -eff_max_loss
786:                        and hold_sec >= profile["min_hold_sec"]):
791:                                    f"(limit={eff_max_loss}%)")
801:                if pos.pnl_pct <= -eff_max_loss and not pos.trail_active:
```

### Main Monitoring Loop (lines 720-800, post-heartbeat shift)

Post-Commit 4 runner now has heartbeat injected at lines 613-623. Real exit/monitor logic has shifted by +10 lines. Showing the relevant window after shift:

```python
                            )
                        continue

            # ── 2. Monitor active positions ──
            for coin in list(positions.keys()):
                pos = positions[coin]
                price = get_price(pos.pair)
                if price <= 0:
                    continue

                pos.update(price)
                hold_sec = time.time() - pos.opened_at

                # Fix 5: Desync detection on every price check cycle
                _verify_position_qty(agent_id, pos)

                # Trail activation
                if (pos.pnl_pct >= profile["trail_activate_pct"]
                        and not pos.trail_active):
                    pos.trail_active = True
                    log.info(f"[{agent_id}] TRAIL ON {coin} | "
                             f"pnl={pos.pnl_pct:+.2f}%")

                # Trail stop: drop from high
                if pos.trail_active:
                    drop = pos.high_pnl_pct - pos.pnl_pct
                    if (drop >= profile["trail_drop_pct"]
                            and hold_sec >= profile["min_hold_sec"]):
                        if pos.pnl_pct > 0:
                            _sell(agent_id, pos, positions, stats,
                                  f"Trail stop: dropped {drop:.2f}% "
                                  f"from high {pos.high_pnl_pct:+.2f}%")
                            continue
                        else:
                            _sell(agent_id, pos, positions, stats,
                                  f"Trail breakeven: was +{pos.high_pnl_pct:.2f}%, "
                                  f"now {pos.pnl_pct:+.2f}%")
                            continue

                # BUG#4 FIX: Multi-level distress with cooldown
                # Use per-position max_loss_override if set (e.g., RISKY-WL entries)
                eff_max_loss = (pos.max_loss_override
                                if pos.max_loss_override > 0
                                else profile["max_loss_pct"])
                if (pos.pnl_pct <= -eff_max_loss
                        and not pos.trail_active
                        and hold_sec >= profile["min_hold_sec"]):
                    cooldown_until = distress_cooldown.get(coin, 0)
                    if now > cooldown_until:
                        log.warning(f"[{agent_id}] DISTRESS {coin} | "
                                    f"pnl={pos.pnl_pct:+.2f}% "
                                    f"(limit={eff_max_loss}%)")
                        db_send_signal(agent_id, "DISTRESS", coin, {
                            "entry_price": pos.entry_price,
                            "current_price": price,
                            "pool": pos.pool,
                            "pnl_pct": pos.pnl_pct,
                        })
                        distress_cooldown[coin] = now + 120  # 2 min cooldown

                # Check B2 distress response
                if pos.pnl_pct <= -eff_max_loss and not pos.trail_active:
                    resp = db_check_distress_response(agent_id, coin)
                    if resp.get("decision") == "CUT":
                        _sell(agent_id, pos, positions, stats,
                              f"B2 CUT: {resp.get('reason','distress')}")
                        distress_cooldown.pop(coin, None)
                        continue

            # ── 3. Periodic DB sync (every 30s) ── BUG#2 FIX
            if now - last_db_sync > 30:
                for pos in positions.values():
                    _sync_position_to_db(pos)
                last_db_sync = now

            # ── 4. Heartbeat log (every 60s) ──
            if now - last_heartbeat > 60 and positions:
                parts = []
                for p in positions.values():
                    trail_tag = " [TRAIL]" if p.trail_active else ""
                    parts.append(f"{p.coin} {p.pnl_pct:+.2f}%{trail_tag}")
```


---

## PART 5: PROFILES Dict (Exit Parameters Per Agent)

### Grep — PROFILES references

```
106:PROFILES = {
111:        "trail_activate_pct": 1.2,   # activate trail after +1.2% (fee-safe: net +0.35% after 0.3% RT fees + 0.15% spread)
112:        "trail_drop_pct":     0.4,   # sell if drops 0.4% from high (was 0.25%)
113:        "max_loss_pct":       1.5,   # distress signal at -1.5% (unchanged)
114:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 120s)
121:        "trail_activate_pct": 1.5,   # activate trail after +1.5% (was +1.0%)
122:        "trail_drop_pct":     0.6,   # sell if drops 0.6% from high (was 0.5%)
123:        "max_loss_pct":       3.0,   # distress signal at -3.0% (unchanged)
124:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 60s)
131:        "trail_activate_pct": 3.0,   # activate trail after +3.0% (was +2.0%)
132:        "trail_drop_pct":     1.2,   # sell if drops 1.2% from high (was 1.0%)
133:        "max_loss_pct":       5.0,   # distress signal at -5.0% (unchanged)
134:        "min_hold_sec":       7200,  # 2 hours minimum hold (was 30s)
581:    log.info(f"  Trail: activate +{profile['trail_activate_pct']}%, "
582:             f"drop {profile['trail_drop_pct']}%")
583:    log.info(f"  Max loss: {profile['max_loss_pct']}% | "
757:                if (pos.pnl_pct >= profile["trail_activate_pct"]
766:                    if (drop >= profile["trail_drop_pct"]
767:                            and hold_sec >= profile["min_hold_sec"]):
783:                                else profile["max_loss_pct"])
786:                        and hold_sec >= profile["min_hold_sec"]):
```

### PROFILES dict (from module import)

```json
{
  "A1": {
    "label": "Stable",
    "max_per_trade_pct": 0.8,
    "max_positions": 2,
    "trail_activate_pct": 1.2,
    "trail_drop_pct": 0.4,
    "max_loss_pct": 1.5,
    "min_hold_sec": 7200,
    "monitor_sec": 5
  },
  "A2": {
    "label": "Volatile",
    "max_per_trade_pct": 0.5,
    "max_positions": 2,
    "trail_activate_pct": 1.5,
    "trail_drop_pct": 0.6,
    "max_loss_pct": 3.0,
    "min_hold_sec": 7200,
    "monitor_sec": 3
  },
  "A3": {
    "label": "Trending",
    "max_per_trade_pct": 0.7,
    "max_positions": 1,
    "trail_activate_pct": 3.0,
    "trail_drop_pct": 1.2,
    "max_loss_pct": 5.0,
    "min_hold_sec": 7200,
    "monitor_sec": 3
  }
}
```


---

## PART 6: tk_fills_raw Summary (v2 period)

**Schema note:** `time` is unix epoch ms (INTEGER), no `created_at` col. Using `DATE(time/1000, 'unixepoch')` to derive day, no `side` col (derived from `is_buyer`).

```
day         symbol    side  fills  total_qty  total_idr   first_fill           last_fill          
----------  --------  ----  -----  ---------  ----------  -------------------  -------------------
2026-04-10  ARB_IDR   BUY   3      180.8      346594.2    2026-04-10 04:52:44  2026-04-10 05:54:00
2026-04-10  ARB_IDR   SELL  3      180.4      341973.2    2026-04-10 05:09:31  2026-04-10 22:21:50
2026-04-10  BTC_IDR   BUY   17     0.00092    1146059.46  2026-04-10 04:52:41  2026-04-10 22:25:09
2026-04-10  BTC_IDR   SELL  1      0.0003     370183.23   2026-04-10 05:09:30  2026-04-10 05:09:30
2026-04-10  DOGE_IDR  BUY   5      309.0      494149.0    2026-04-10 01:24:35  2026-04-10 14:27:21
2026-04-10  DOGE_IDR  SELL  6      308.0      494827.0    2026-04-10 03:23:48  2026-04-10 19:39:07
2026-04-10  ETH_IDR   BUY   4      0.004      150996.58   2026-04-10 04:52:41  2026-04-10 13:27:14
2026-04-10  ETH_IDR   SELL  3      0.004      151366.11   2026-04-10 05:09:31  2026-04-10 14:27:16
2026-04-10  SOL_IDR   BUY   3      0.4138     592177.07   2026-04-10 02:15:03  2026-04-10 13:54:43
2026-04-10  SOL_IDR   SELL  3      0.4129     594501.54   2026-04-10 04:39:18  2026-04-10 20:41:05
2026-04-10  SUI_IDR   BUY   1      5.49       88290.18    2026-04-10 13:54:43  2026-04-10 13:54:43
2026-04-10  SUI_IDR   SELL  1      3.87       61447.86    2026-04-10 02:03:04  2026-04-10 02:03:04
2026-04-10  TKO_IDR   BUY   2      201.8      193994.77   2026-04-10 04:52:47  2026-04-10 05:13:05
2026-04-10  TKO_IDR   SELL  2      201.4      195198.99   2026-04-10 05:09:31  2026-04-10 13:45:53
2026-04-10  WLD_IDR   BUY   3      70.01      323414.49   2026-04-10 04:52:44  2026-04-10 05:13:08
2026-04-10  WLD_IDR   SELL  2      69.86      321285.54   2026-04-10 05:09:31  2026-04-10 13:45:53
2026-04-11  SOL_IDR   BUY   1      0.0144     20893.68    2026-04-11 00:44:26  2026-04-11 00:44:26
```

### tk_fills_raw count overview

```
earliest             latest               total_fills  buys  sells
-------------------  -------------------  -----------  ----  -----
2026-04-05 07:27:59  2026-04-11 00:44:26  157          89    68   
```


---

---

## PART 7: Price Evolution During Longest Hold (A3 SUI #17)

**Context:** Position #17 A3 SUI opened 2026-04-10 13:54 UTC, closed 2026-04-12 12:36 UTC.
Hold = 2801.78 min ≈ **46.7 jam**. Entry 16,065.5 IDR, Sell 15,569.5 IDR, PnL -0.03%.
Fetch 1h klines untuk SUI_USDT (bersih dari zero-vol IDR) selama window ~48h.

```
# Available kline/ohlcv functions: ['TOKO_MBX_KLINES', 'get_ohlcv', 'get_order_history']
# Method used: tk.get_ohlcv('SUI_USDT', '60', 80)
# Rows fetched: 80, columns: ['time', 'open', 'high', 'low', 'close', 'volume']
# 'time' dtype: datetime64[ms]
# Window [2026-04-10 13:54:00 → 2026-04-12 12:36:00] rows: 36
# Entry IDR = 16,065.5, approx USDT ≈ 0.9620

utc_time                   open       high        low      close       volume  pct_vs_entry
------------------------------------------------------------------------------------------------
2026-04-11 01:00        0.93900    0.94020    0.93510    0.93860    505329.90        -2.43%
2026-04-11 02:00        0.93860    0.94310    0.93760    0.94060    426859.20        -2.23%
2026-04-11 03:00        0.94070    0.94490    0.93650    0.93700    633737.70        -2.60%
2026-04-11 04:00        0.93700    0.93960    0.93560    0.93770    402434.30        -2.53%
2026-04-11 05:00        0.93770    0.93830    0.93230    0.93320    646406.50        -2.99%
2026-04-11 06:00        0.93320    0.93340    0.92960    0.93180    798233.80        -3.14%
2026-04-11 07:00        0.93190    0.93430    0.93090    0.93330    695006.90        -2.98%
2026-04-11 08:00        0.93330    0.93750    0.93200    0.93300    443787.20        -3.02%
2026-04-11 09:00        0.93290    0.93760    0.93190    0.93650    718747.10        -2.65%
2026-04-11 10:00        0.93640    0.93720    0.93330    0.93480    769415.20        -2.83%
2026-04-11 11:00        0.93470    0.93600    0.93300    0.93340    557620.80        -2.97%
2026-04-11 12:00        0.93330    0.93560    0.93090    0.93340    475426.00        -2.97%
2026-04-11 13:00        0.93340    0.93340    0.92720    0.93230    879527.40        -3.09%
2026-04-11 14:00        0.93230    0.93450    0.93040    0.93150    383126.20        -3.17%
2026-04-11 15:00        0.93150    0.93680    0.93100    0.93600    378036.20        -2.70%
2026-04-11 16:00        0.93610    0.95140    0.93590    0.94650   1564984.20        -1.61%
2026-04-11 17:00        0.94650    0.94730    0.94320    0.94480    677630.10        -1.79%
2026-04-11 18:00        0.94490    0.95910    0.94170    0.95350   1683607.10        -0.88%
2026-04-11 19:00        0.95350    0.96300    0.95240    0.96240   1823011.00        +0.04%
2026-04-11 20:00        0.96240    0.96380    0.95070    0.95280   1869808.50        -0.96%
2026-04-11 21:00        0.95280    0.95740    0.94890    0.95370    708681.00        -0.86%
2026-04-11 22:00        0.95370    0.95850    0.94630    0.94840   1990272.80        -1.41%
2026-04-11 23:00        0.94830    0.94830    0.93780    0.94150    756557.20        -2.13%
2026-04-12 00:00        0.94160    0.94160    0.93690    0.94130    818554.60        -2.15%
2026-04-12 01:00        0.94120    0.94390    0.90620    0.91170   4684122.10        -5.23%
2026-04-12 02:00        0.91160    0.91730    0.90560    0.91450   2493095.60        -4.94%
2026-04-12 03:00        0.91450    0.91700    0.91050    0.91230    974130.90        -5.17%
2026-04-12 04:00        0.91230    0.91310    0.90980    0.91140    640442.90        -5.26%
2026-04-12 05:00        0.91140    0.91560    0.90770    0.91490    775762.70        -4.90%
2026-04-12 06:00        0.91480    0.91670    0.91210    0.91290    623110.70        -5.10%
2026-04-12 07:00        0.91290    0.91400    0.91060    0.91260    780447.80        -5.14%
2026-04-12 08:00        0.91260    0.91420    0.91100    0.91230    363969.70        -5.17%
2026-04-12 09:00        0.91240    0.91330    0.91100    0.91210    336292.10        -5.19%
2026-04-12 10:00        0.91210    0.91300    0.90620    0.90760    518943.70        -5.66%
2026-04-12 11:00        0.90750    0.91220    0.90660    0.91110    427491.50        -5.29%
2026-04-12 12:00        0.91100    0.91390    0.90650    0.90740    835524.50        -5.68%

# Window summary:
#   candles:           36
#   min(low):          0.90560
#   max(high):         0.96380
#   open(first):       0.93900
#   close(last):       0.90740
#   total_range_pct:   6.43%
```

---

## PART 8: PM2 Logs Excerpt — A3 SUI Window (2026-04-10 → 2026-04-12)

### Available A3 log files

```
-rw-r--r-- 1 root root 0 Apr 10 12:13 /root/.pm2/logs/aria-A3-out.log
```

### SUI-related log lines from A3 (all rotations)

```
--- /root/.pm2/logs/aria-A3-out.log (0
0 matches) ---

Grep SUI content across all A3 logs (limit 60 lines):
```

### Raw log sample (aria-A3-out.log last 30 lines if exists)
```
```

