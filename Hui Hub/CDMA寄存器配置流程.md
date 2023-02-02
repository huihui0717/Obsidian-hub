### 1. CDMA Program Guild
![[Pasted image 20230202205837.png]]
reference CDMA spec

### 2. CDMA Verification CODE
#### 2.1  总流程

```verilog
task DoCdmaWork();

    get_ddr_reg_value(addrmap,itlv, aid);   // 获取DDR interleave 参数 

    CDMASetUp();          // step 1 配置0x800寄存器，使能CDMA
        
    GetSimParameters();

    ConstructDescriptorChain();   // 生成 descriptor 数据

    fork
            
        begin
            ConfigDescriptorByAXI();    // step2 配置0x878~0x894寄存器
        end
             
        begin
            for(int unsigned i=0; i<deschain_length; i++) begin
  
                ConstructSourceData(i);   // 生成原始数据
                    
                WaitForOneDescriptorDone(i);  // step3  等待CDMA完成一次数据搬运
                    
                DoDataCompare(i);

                FreeDataMem(i);

                `uvm_info(get_name(), $sformatf("Total %0d descriptors, %0d has been processed...",deschain_length,i+1),UVM_LOW)
            end
        end
            
    join
    
    #1000ns;
    
    CDMAShutDown();          // step 4   关闭CDMA

        
endtask: DoCdmaWork
```

### 2.2 各个task
#### 2.2.1 CDMASetUp
```verilog
task pcie_cdma_test_seq::CDMASetUp();

    bit[31:0]           reg_wdata;
    bit[31:0]           reg_rdata;
        
    ctrl_reg = cdma_ctrl_reg::type_id::create("ctrl_reg");

    assert(ctrl_reg.randomize());

    ctrl_reg.print();

    reg_wdata = {ctrl_reg.chl_resv2, ctrl_reg.write_qos, ctrl_reg.read_fixed_qos, ctrl_reg.read_dyn_low_qos, ctrl_reg.read_dyn_normal_qos, ctrl_reg.read_dyn_high_qos, ctrl_reg.read_qos_mode, ctrl_reg.chl_resv1, ctrl_reg.int_out_en, ctrl_reg.chl_resv0, ctrl_reg.data_dma_en};      
        
    `uvm_info(get_name(),$sformatf("CDMA int_enable is %0b, new descriptor is started",ctrl_reg.int_out_en),UVM_LOW)
        
    reg32_write(sub_env_cfg.cdma_base_addr+`CDMA_CTRL_ADDR, reg_wdata);
    reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_CTRL_ADDR,reg_rdata);

    if(reg_rdata != reg_wdata) begin
        `uvm_error(get_name(),$sformatf("CDMA CTRL_REG read check fail!!, wdata: %0h, rdata: %0h", reg_wdata, reg_rdata));
    end

    if(ctrl_reg.int_out_en == 1'b1) begin
         if(ctrl_reg.rf_cmd_ept_int_mask==1'b1) begin
            if(sub_env_cfg.pcie_misc_vif.cdma_intr == 1'b1)
                `uvm_error(get_name(), "Interrupt pin should be low when cdma shutdown interrupt out in just config ctrl reg!")
        end
        else begin
            if(sub_env_cfg.pcie_misc_vif.cdma_intr == 1'b0)
                `uvm_error(get_name(), "Interrupt pin should be high when cdma setup interrupt out in just config ctrl reg!")
        end
    end
    else begin
        if(sub_env_cfg.pcie_misc_vif.cdma_intr == 1'b1)
            `uvm_error(get_name(), "Interrupt pin should be low when cdma int_out_en is 0")
    end

    reg_wdata = {ctrl_reg.max_pay_resv0, ctrl_reg.write_max_payload, ctrl_reg.read_max_payload}; 
    reg32_write(sub_env_cfg.cdma_base_addr+`CDMA_MAX_PAYLOAD, reg_wdata);
    reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_MAX_PAYLOAD,reg_rdata);

    `uvm_info(get_name(), $sformatf("CDMA MAX_PAYLOAD REG wdata: %0h, rdata: %0h", reg_wdata, reg_rdata), UVM_L  OW)
     if(reg_rdata != reg_wdata) begin
        `uvm_error(get_name(),$sformatf("CDMA MAX_PAYLOAD REG read check fail!!"))
    end
          
endtask: CDMASetUp
```
##### 2.2.2 ConfigDescriptorByAXI
```verilog
task ConfigDescriptorByAXI();

    bit[31:0]   data;
    bit[31:0]   rdata;
    bit[255:0] des_bits;

    while(pio_used_des < deschain_length) begin
            
        int des_index = this.pio_used_des;            
            
        reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_FIFO_STATUS, data);   

        while(data > 7) begin
            #500;
            reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_FIFO_STATUS, data);
            //$display("When CDMA_FIFO_STATUS greater than 7, current cmd_fifo: %0d", data);
        end


        des_chain[des_index].print();
        des_bits = des_chain[des_index].pack_des();
            
        for(int unsigned i=8;i>0;i--) begin  // a descriptor is 80 bytes in size, and descp_vld(des_bits[0]) must be written at last
            reg32_write(sub_env_cfg.cdma_base_addr+`PIO_BUF_ADDR+(i-1)*4, des_bits[(i-1)*32+:32], 1);
            reg32_read(sub_env_cfg.cdma_base_addr+`PIO_BUF_ADDR+(i-1)*4, rdata);
        end
        this.pio_used_des++;

        `uvm_info(get_name(), $sformatf("cmd_fifo descriptot: %0d", pio_used_des), UVM_LOW);

     end
        
     `uvm_info(get_name(), "ConfigDescriptorByAXI done!", UVM_LOW)
    
endtask: ConfigDescriptorByAXI
```
##### 2.2.3 WaitForOneDescriptorDone
```verilog
task pcie_cdma_test_seq::WaitForOneDescriptorDone(int des_index);
                
    bit[31:0]       wdata;
    bit[31:0]       rdata;
    bit[31:0]       cmd_fifo;
    bit             cmd_done;
    bit             cdma_intr; 
        
    `uvm_info(get_name(), $sformatf("Start %0dth WaitForOneDescriptorDone", des_index), UVM_LOW)
    
    WaitDesDone(cdma_done_time);

    ComputeBandwidth(des_index);
         
    fork

        begin
            #50ns;
            cdma_start_time = $realtime();
        end

        begin
             
           cmd_done = sub_env_cfg.pcie_misc_vif.cdma_done;  //use interface instead of read axi ,  to save time , sync descriptor done at the first time            

            if(des_chain[des_index].eod^cmd_done) begin
                `uvm_error(get_name(),$sformatf("CDMA Interrupt status not set after descriptor finish."))
            end          

            cmd_fifo = 32'h1;        
            reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_FIFO_STATUS, cmd_fifo);

            cdma_intr = sub_env_cfg.pcie_misc_vif.cdma_intr;

            reg32_read(sub_env_cfg.cdma_base_addr+`CDMA_INT_STATUS, rdata);    
            if(ctrl_reg.int_out_en == 1'b0 || ((ctrl_reg.rf_des_int_mask == 1'b1 || des_chain[des_index].int_en == 1'b0) && ctrl_reg.rf_cmd_ept_int_mask == 1'b1)) begin    // must no interrupt output in pin 
                if(des_chain[des_index].int_en == 1'b1) begin
                        
                    if((cmd_fifo == 0 && rdata[3] == 0) || (cmd_fifo != 0 && rdata[3] == 1))
                        `uvm_error(get_name(), $sformatf("CDMA cmd fifo empty intrrupt in error mode!, cmd_fifo: %0d, cmd_ept_int: %0d", cmd_fifo, rdata[3]))
                        
                    if(rdata[0] == 1'b0)
                        `uvm_error(get_name(), "Descriptor intrrupt should generate when des_int_en enable!")
                        
                    if(cdma_intr == 1'b1)
                        `uvm_error(get_name(), "Interrupt pin should be low when mask intr_out!")                        
                end
                else begin
                    if((cmd_fifo == 0 && rdata[3] == 0) || (cmd_fifo != 0 && rdata[3] == 1))                        
                        `uvm_error(get_name(), $sformatf("CDMA cmd fifo empty intrrupt in error mode!, cmd_fifo: %0d, cmd_ept_int: %0d", cmd_fifo, rdata[3]))
                        
                    if(rdata[0] == 1'b1)
                        `uvm_error(get_name(), $sformatf("Descriptor intrrupt should not generate when des_int_en disable!, %0dth", des_index))
                        
                    if(cdma_intr == 1'b1)
                        `uvm_error(get_name(), "Interrupt pin should be low when mask intr_out!")
                end
            end
            else begin          // maybe have or no interrupt output
                if(des_chain[des_index].int_en == 1'b1) begin
                        
                    if((cmd_fifo == 0 && rdata[3] == 0) || (cmd_fifo != 0 && rdata[3] == 1))
                        `uvm_error(get_name(), $sformatf("CDMA cmd fifo empty intrrupt in error mode!, cmd_fifo: %0d, cmd_ept_int: %0d", cmd_fifo, rdata[3]))

                    if(rdata[0] == 1'b0)
                        `uvm_error(get_name(), "Descriptor intrrupt should generate when des_int_en enable!")

                    if(cdma_intr == 1'b0) begin     // no interrupt output
                        if(ctrl_reg.int_out_en == 1'b1 && ((ctrl_reg.rf_des_int_mask == 1'b0) || (rdata[3]==1 && ctrl_reg.rf_cmd_ept_int_mask == 1'b0))) begin
                            `uvm_error(get_name(), "Interrupt pin should be high when ctrl_reg.int_out_en == 1'b1 && ((ctrl_reg.rf_des_int_mask == 1'b0) || (rdata[3]==1 && ctrl_reg.rf_cmd_ept_int_mask == 1'b0))!") 
                         end
                    end
                    else begin  // have interrupt output
                        if(ctrl_reg.int_out_en == 1'b0 || ((ctrl_reg.rf_des_int_mask == 1'b1) && (ctrl_reg.rf_cmd_ept_int_mask == 1'b1))) begin
                            `uvm_error(get_name(), "Interrupt pin should be low when ctrl_reg.int_out_en == 1'b0 || ((ctrl_reg.rf_des_int_mask == 1'b1) && (ctrl_reg.rf_cmd_ept_int_mask == 1'b1))!") 
                        end
                    end
                end
                           
                                
```