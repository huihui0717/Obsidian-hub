## 一、功能点
### 1.1  1D
- data_length
> data_length 最大支持(2^32-1)，只支持 8bit。

### 1.2 2D
- data_fmt
> 8bit 或 16bit，当data_fmt = 16bit，地址需要对齐
- const_en
> 2'b00: no padding；2‘b01: paddinf fixed value
- image_height, image_width, src_wstride, dst_wstride
> all inside [1 : 2^16-1]
> src_wstride >= image_width
> dst_wstride >= image_width
## 二、1D
### 2.1 测试点
1D数据时，只有一个测试点 data_length，需要测试边界条件即，data_length=1 和 data_length=2^32-1。
### 2.2 测试用例
sim.cfg 中传递仿真参数，控制不同的仿真方案
#### 2.2.1 tc_cdma_1D_pio_user_specify_mode_test
该模式下，用户可以直接配置data_length的值，可以测试边界条件
 - user_mode
 > 该参数控制用户模式，当 user_mode=1时，用户可以直接指定 CDMA FIFO descriptor 的配置。
- user_data_len
>该参数在用户模式下，可以指定 data_length 的值。
>**testcase name**: tc_cdma_1D_pio_user_specify_mode_test
#### 2.2.2 tc_cdma_1D_pio_short_length_mode_test
该用例测试 data_length 短模式即([1 : 2^25-1])
#### 2.2.3 tc_cdma_1D_pio_long_length_mode_test
该用例测试 data_le



### 普通模式
- data_len_mode
> 该参数可以控制 data_length 的取值范围

> data_len_mode=0    短数据模式：data_length inside \[1 :  2^25-1\]. 
    **testcase name**: tc_cdma_1D_pio_short_length_mode_test

> data_len_mode=1    长数据模式：data_length inside \[2^25 : 2^32-1\]
> **testcase name** :   tc_cdma_1D_pio_long_length_mode_test

## 三、2D
sim.cfg 中传递仿真参数，控制不同的仿真方案
### 用户模式
- user_mode
>该参数控制用户模式
- user_data_fmt
>控制像素单位，8bit 和 16bit
- user_const_en
>控制padding
- user_image_width
>2D图像的宽度
- user_image_height
>2D图像的高度
- user_src_wstride
>源图像的步幅
- user_dst_wstride
>目的图像的步幅

**testcase**: tc_cdma_2D_pio_user_specify_mode_test

### 普通模式
- data_fmt_mode
> 控制数据的格式，8bit和16bit
- padding_mode
> 控制 padding，是否padding 固定值
- image_len_mode
> 控制图像长度模式，短模式和长模式

**testcase**: 
- tc_cdma_2D_pio_8bit_padding_short_test
- tc_cdma_2D_pio_8bit_padding_long_test
- tc_cdma_2D_pio_8bit_no_padding_short_test
- tc_cdma_2D_pio_8bit_no_padding_long_test
- tc_cdma_2D_pio_16bit_padding_short_test
- tc_cdma_2D_pio_16bit_padding_long_test
- tc_cdma_2D_pio_16bit_no_padding_short_test
- tc_cdma_2D_pio_16bit_no_padding_long_test



