1. regBuilder -nogui -sheet_name Reg_Table -e ./SG2260_c2c_Top_Reg_v1.1.xlsx -o ./result -doc -sv_ralf
2. bsub -Is -q dv_regression ralgen -t c2c_top_reg -I ./ -uvm ./reg_c2c_top_reg.ralf


top.u_dut_top.u_dut.chip_core.u_peri_sys.u_peri.DW_apb_timers.timer_en[7:0]