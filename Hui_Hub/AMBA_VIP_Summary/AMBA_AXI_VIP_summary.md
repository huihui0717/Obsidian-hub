
### AXI outstanding 模式
配置相关参数：
	1. svt_axi_port_configuration
		1. num_outstanding_xact、num_read_outstanding_xact、num_write_outstanding_xact
>指定主机/从机可以支持的未完成事务数。AXI4_STREAM不支持num_outstanding_xact=-1
MASTER：如果未完成交易的数量等于此数量，则在未完成交易数量小于此参数之前，主机将禁止启动任何新交易。
SLAVE：如果未完成事务的数量等于此数量，则在未完成事务数量小于此参数之前，从机将不会断言ARREADY/AWREADY。
如果num_outstanding_xact=-1，则不会考虑num_unstanding_xact，而会影响num_read_outstanting_xact和num_write_outstaning_xact。

配置 config_sequence中的 wait_response=0