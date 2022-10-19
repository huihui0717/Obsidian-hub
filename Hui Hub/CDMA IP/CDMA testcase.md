## 一、1D
sim.cfg 中传递仿真参数，控制不同的仿真方案
### 用户模式
 - user_mode
 > 该参数控制用户模式，当 user_mode=1时，用户可以直接指定 CDMA FIFO descriptor 的配置。
- user_data_len
>该参数在用户模式下，可以指定 data_length 的值。
>**testcase name**: tc_cdma_1D_pio_user_specify_mode_test

### 普通模式
- data_len_mode
> 该参数可以控制 data_length 的取值范围

> data_len_mode=0    短数据模式：data_length inside \[1 :  2^25-1\]. 
    **testcase name**: tc_cdma_1D_pio_short_length_mode_test

> data_len_mode=1    长数据模式：data_length inside \[2^25 : 2^32-1\]
> **testcase name** :   tc_cdma_1D_pio_long_length_mode_test

## 二、2D
sim.cfg 中传递仿真参数，控制不同的仿真方案

