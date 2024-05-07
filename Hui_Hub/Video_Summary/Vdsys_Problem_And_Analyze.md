1. vdsys_pr_wrap.v
	1.  AXI Master width configuration
		 id_width             8bit
		 addr_width         32bit
		 len_width            4bit
		 	       



从compile.log 提取和记录相关Warning 的一般步骤(仅供参考)：
	1、命令：提取file 文件中 Warning所在行及后 m 行的内容，并保存在 newfile 中：sed -n '/Warning/, +mp' file > newfile;
    2、从newfile中记录具体某类Waring;	
    3、删除步骤二中的Warning, ，命令：删除newfile 文件中 XXX所在行及后 m 行的内容，并保存在新文件中：sed  '/XXX/, +md' newfile > newfile01;
    4、重复步骤2-3，直到没有Warning;
