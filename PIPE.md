

```systemverilog
task cover_ram_2p_1c(input string heir_path, input int ram_width, input int ram_depth, input int addr_width);

  string  clk_heir;
  string  rst_heir;
  string  addra_heir;
  string  addrb_heir;
  string  datain_heir;
  string  wea_heir;
  string  ena_heir;
  string  enb_heir;
  string  dataout_heir;

  bit [722-1:0]  datain;
  bit [722-1:0]  dataout;
  
  clk_heir      = {heir_path, "clk"};
  rst_heir      = {heir_path, "rst"};
  addra_heir    = {heir_path, "_addra"};
  addrb_heir    = {heir_path, "_addrb"};
  datain_heir   = {heir_path, "_datain"};
  wea_heir      = {heir_path, "_wea"};
  ena_heir      = {heir_path, "_ena"};
  enb_heir      = {heir_path, "_enb"};
  dataout_heir  = {heir_path, "_dataout"};

  // pull up rst_n
  uvm_hdl_force(rst_heir, 1);

  for(int addr=0; addr < ram_depth; addr++) begin
	uvm_hdl_force(clk_heir, 0);
	#5ns;
	uvm_hdl_force(clk_heir, 1);

    #1ns;
    // force ram_addra
    uvm_hdl_force(addra_heir, addr);
    // pull up wea
    uvm_hdl_force(wea_heir, 32'h1);
    // pull up ena
    uvm_hdl_force(ena_heir, 32'h1);
    // force wdata
    std::randomize(datain);
    uvm_hdl_force(datain_heir, datain);
    #4ns;

	uvm_hdl_force(clk_heir, 0);
	#5ns;
	uvm_hdl_force(clk_heir, 1);
		
	// release ram write signal
    uvm_hdl_release(addra_heir);
    uvm_hdl_release(wea_heir);
    uvm_hdl_release(ena_heir);
    uvm_hdl_release(datain_heir);
    #5ns;

	uvm_hdl_force(clk_heir, 0);
	#5ns;
	uvm_hdl_force(clk_heir, 1);

    #1ns;
    // force read addrb
    uvm_hdl_force(addrb_heir, addr);
    // pull up enb
    uvm_hdl_force(enb_heir, 32'h1);
    #4ns;

	uvm_hdl_force(clk_heir, 0);
	#5ns;
	uvm_hdl_force(clk_heir, 1);
	#5ns;
	uvm_hdl_force(clk_heir, 0);

    // read ram data
    if(!uvm_hdl_read(dataout_heir, dataout))  `uvm_error(get_name(), "UVM HDL READ DOUTB FAILED!")
    // release read addrb
    uvm_hdl_release(addrb_heir);
    // pull down enb
    uvm_hdl_release(enb_heir);
    `uvm_info(get_name(), $sformatf("Ram check addr: %0h, dina: %0h, doutb: %0h", addr, datain, dataout), UVM_NONE)
    for(int j=0; j < ram_width; j++) begin
      if(datain[j] != dataout[j]) `uvm_error(get_name(), $sformatf("Check ram write and read failed!,index: %0d,  dina: %0h, doutb: %0h", j, datain[j], dataout[j]))
    end

	#5ns;
	uvm_hdl_force(clk_heir, 1);
	#5ns;
  end
  uvm_hdl_release(rst_heir);

endtask

```






