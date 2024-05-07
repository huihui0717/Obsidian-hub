
Packages 是一种 System Verilog的语言结构，它使相关的声明和定义能够在Packages空间中组合在一起。Packages可能包含类型定义、常量声明、函数和类模板。

若要在作用域内使用package，则必须使用 import 导入该package，然后才能引用其内容。

Package是组织代码和确保对类型、类等的引用一致的一种有用的方法。这个UVM基类库包含在一个名为"UVM_pkg"的package中。开发UVM测试平台时应该使用package来收集和组织为实现而开发的各种类定义agent、env、sequence libraries、test libraries等。

#### UVM Package 编码指南

##### Package 命名和文件命名约定

Package 应该以 \_pkg 的后缀命名。Package 文件命名应该以反应该package的名字命名，并且具有 .sv 的扩展名。

例如：文件 spi_env_pkg.sv 将包含 spi_env_pkg package。

理由：sv 扩展名是一种约定，表示package文件是一个独立的编译单元。\_pkg 后缀表示该文件包含一个 package。这种约定对人和机器解析脚本都非常有用。

##### Package 中包含的 类 应该被 \`include

在 package 的范围内声明的类模板应该分离为扩展名为 .svh 的单独文件。这些应该按照需要编译的顺序被 \`include 在package中。 Package文件是唯一应该使用 \`include的地方，被include的文件中不应该有其他\`include的语句。

理由：在单独的文件中声明类使它们更容易维护，也更清楚package内容是什么。

##### Import 其他 package的语句应当在package开头部分声明

一个 package 的内容可能需要引用另一个package中的内容。在这种情况下，外部 package 应在 package 的代码体的头部声明。独立文件，如可能被包含的类模板，不应该单独 import。

理由：将所有 import 分组到一个位置可以清楚地了解package的依赖项。将 import 放入 package 的其他部分或包含的文件中可能会导致排序和潜在的类型 

##### Package使用的所有文件都应收集在一个目录中

包含在一个 package 中的所有文件都应该集中在一个目录中。这对于代理目录结构需要为完整独立package的代理尤其重要。

理由：这使得编译更容易，因为只有一个包含目录，它还有助于重用，因为可以轻松地将包的所有文件收集在一起。

下面是 UVM 环境的软件包文件示例。此 env 包含两个代理（spi 和 apb）和一个寄存器模型，它们作为子包导入。与环境相关的类模板包括：
```systemverilog
// Note that this code is contained in a file called spi_env_pkg.sv
//

// 
// Package Description
// 

package spi_env_pkg;

// Standard UVM import & include
import uvm_pkg::*;
`include "uvm_macros.svh"


// Any further package imports:
import apb_agent_pkg::*;
import spi_agent_pkg::*;
import spi_register_pkg::*;


// Includes:
`include "spi_env_config.svh"
`include "spi_virtual_sequencer.svh"
`include "spi_env.svh"

endpackage: spi_env_pkg
```

##### Package Scopes

经常让用户感到困惑的是，SystemVerilog Package 是一个范围。这意味着包中声明的所有内容以及导入包中的其他包的内容仅在包范围内可见。如果将包导入到另一个范围（即另一个包或模块），则只有包的内容可见，而看不到它导入的任何包的内容。如果新范围中需要这些其他包的内容，则需要单独导入它们。
```systemverilog
// 
// Package Scope Example
// -------------------------------------
// 

package spi_test_pkg;

// The UVM package has to be imported, even though it is imported in the spi_env
// package. This because the import of the uvm_pkg is only visible within the 
// current scope

import uvm_pkg::*;
// The same is ture of the `include of the uvm_macros
`include "uvm_macros.svh"

// Import of uvm_pkg inside the spi_env package is not 
// visible within the scope of the spi_test package 

import spi_env_pkg::*; 

// Other imports 
// Other `includes 
`include spi_test_base.svh

endpackage: spi_test_pkg
```