### 1 PHY 的使用和配置
#### 1.1 PHY Powr-On(上电)
PHY 允许按任何顺序接通 vp* 和 vph 电源，然后进行 PHY 的复位和初始化序列(vp* 表示所有核心电源组；vp、vpdig和vptxN，其中N是通道数 )。

**注意:**
		- PHY 在上电后但在复位前的时间段内的状态是**未知**的。PHY 可能会出现用户不想要的行为，如：启用TX或激活RX终端。
		- 虽然未知状态是不可取的，但PHY已被设计为在保持这种状态时**阻止**其破坏行为。
		- 电源斜坡(上升/下降)应是单调的，斜坡时间不超过10µs，速率不超过（电源电压）/10µs。
		- vddcore 在 ana_pwr_stable 之前 ramp up(斜坡上升)是**支持**的；ana_pwr_stable在vddcore 之前 ramp down (斜坡下降)是**支持**的。
		- ana_pwr_stable 在 vddcore 之前 ramp up 是**不支持**的；vddcore 在 ana_pwr_stable 之前 ramp down 是**不支持**的；
		- vddcore 和 ana_pwr_strable 一起 ramp up/down 是**支持**的；





未完成Task:
-  chip_id_cfg 没找到相应的寄存器
- pipe_mode -> ss_mode 只1位有效
-  exload_phy_cfg -> fw?, 
- config_ctrl  -> 配置 speed 寄存器未找到
- app_clk_req_cfg
- link_cfg -> DBI寄存器


// 后续在说
- mps_cfg
- space_reg_cfg
- atu_for_ep_cfg




top.u_dut_top.u_dut.u_sg2260_core.u_c2c_sys_0_pr_wrap.c2c_pr_wrap.u_c2c_sys.u_pcie_x16_wrapper.u_32g_phy0_rom.phy_32g_rom_wrapper.u_mem_u_phy_32g_rom_0_0.