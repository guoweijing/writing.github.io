---
title: 流水线CPU设计
categories:
- architecture
tag :

---   

# 实验五：流水线CPU

[toc]

## 实验5-1：流水线处理器集成

### 1.实验目的

​	1.理解流水线CPU的基本原理和组织结构
​	2.掌握五级流水线的工作过程和设计方法
​	3.理解流水线CPU停机的原理
​	4.设计流水线测试程序

### 2.实验任务

#### 任务一：集成设计流水线CPU，在Exp04的基础上完成

​	利用五级流水线各级封装模块集成CPU替换 Exp04的单周期CPU为本实验集成的五级流水线CPU

#### 任务二：设计流水线测试方案并完成测试	

### 3.实验流程

#### 3.1 Pipeline_IF

- ​	流水线CPU第一阶段
- ​	根据程序计数器从指令存储器中取出指令

![image-20230612103334776](https://gitee.com/guoweijing/photo/raw/master/202306121033500.png)

1. ​	由程序计数器获取PC值及PC值的更新
2. ​	由PC值获取指令

```verilog
module Pipeline_IF( 
input [31:0]inst_in_IF,
output reg[31:0]inst_out_IF,
input clk_IF, //时钟
input rst_IF, //复位
input en_IF, //使能
input [31:0] PC_in_jalr,
input [31:0] PC4_in_IF,
input [31:0] PC_in_IF, //取指令PC输入
input [1:0]PCSrc, //PC输入选择
input NOP_IFID,
output  reg[31:0] PC_out_IF //PC输出
); 
wire [31:0] new_pc;
MUX4T1 MUX4T1_0(
.s(PCSrc),
.I0(PC4_in_IF),
.I1(PC_in_IF),
.I2(PC_in_jalr),
.I3(0),//PCSrc不会等于4的
.o(new_pc));
```



#### 3.2 Pipeline—IF_reg_ID    

- ​	流水线CPU取指和译码之间的寄存器
- ​	存储PC值和指令

![image-20230612103826819](https://gitee.com/guoweijing/photo/raw/master/202306121038888.png)

​	寄存IF级的输出指令，分割IF级和ID级的指令或控制信号，防止相互干扰，在IF级执行结束时将指令的控制信号传递至下一级。

```verilog
module IF_Reg_ID( 
input clk_IFID, //寄存器时钟
input rst_IFID, //寄存器复位
input en_IFID, //寄存器使能
input [31:0] PC_in_IFID, //PC输入
input [31:0] inst_in_IFID, //指令输入
output reg [31:0] PC_out_IFID, //PC输出
output reg [31:0] inst_out_IFID, //指令输出
output reg valid_IFID//寄存器有效
);
```



#### 3.3 Pipeline—ID   

- ​	流水线CPU第二阶段
- ​	指令译码

![image-20230612104010520](https://gitee.com/guoweijing/photo/raw/master/202306121040586.png)

​	译码是指将从指令存储器取指的指令进行翻译的过程；译码之后产生各种控制信号，同时寄存器堆根据所需操作数寄存器的索引读出操作数，立即数生成单元输出所需立即数

```verilog
module Pipeline_ID( 
input clk_ID, //时钟
input rst_ID, //复位
input RegWrite_in_ID, //寄存器堆使能
input [4:0] Rd_addr_ID, //写目的地址输入
input [31:0] Wt_data_ID, //写数据输入
input [31:0] Inst_in_ID, //指令输入
output [4:0] Rd_addr_out_ID, //写目的地址输出
output wire [31:0] Rs1_out_ID, //操作数1输出
output wire [31:0] Rs2_out_ID, //操作数2输出
output wire [31:0] Imm_out_ID, //立即数输出
output wire ALUSrc_B_ID, //ALU B端输入选择
output wire [31:0] ALU_control_ID,//ALU控制
output wire Branch_ID, //Branch控制
//output reg BranchN_ID, //Bne控制a
output wire MemRW_ID, //存储器读写
output wire [1:0] Jump_ID, //Jal,Jalr控制
output wire [2:0] MemtoReg_ID, //寄存器写回选择
output wire RegWrite_out_ID
) ;//寄存器堆读写
```



#### 3.4 Pipeline—ID_reg_Ex    

- ​	流水线CPU译码和执行之间的寄存器
- ​	存储ALU数据和控制信号

![image-20230612104211571](https://gitee.com/guoweijing/photo/raw/master/202306121042685.png)

​	寄存ID级的输出指令，分割ID级和EX级的指令或控制信号，防止相互干扰，在ID级执行结束时将指令的控制信号传递至下一级.

```verilog
module ID_Reg_EX(
input clk_IDEX,
input rst_IDEX,
input en_IDEX,
input NOP_IDEX,
input valid_in_IDEX,
input [31:0]PC_in_IDEX,
input [31:0]inst_in_IDEX,
//input [31:0]Wt_addr_IDEX,
input [31:0]Rs1_in_IDEX,
input [31:0]Rs2_in_IDEX,
input [31:0]Imm_in_IDEX,
input ALUSrc_B_in_IDEX,
input [31:0] ALU_control_in_IDEX,
input Branch_in_IDEX,
input MemRW_in_IDEX,
input [1:0]Jump_in_IDEX,
input [2:0]MemtoReg_in_IDEX,
input RegWrite_in_IDEX,
input [4:0] Rd_addr_IDEX,
output reg[31:0]inst_out_IDEX,
output reg[31:0]PC_out_IDEX,
//output reg[4:0]Wt_addr_out_IDEX,
output reg[31:0] Rs1_out_IDEX,
output reg[31:0] Rs2_out_IDEX,
output reg[31:0] Imm_out_IDEX,
output reg ALUSrc_B_IDEX,
output reg[31:0]ALU_control_out_IDEX,
output reg Branch_out_IDEX,
output reg[1:0] Jump_out_IDEX,
output reg MemRW_out_IDEX,
output reg RegWrite_out_IDEX,
output reg [2:0]MemtoReg_out_IDEX,
output reg [4:0] Rd_addr_out_IDEX,
output reg valid_out_IDEX
    );
```



#### 3.5 Pipeline—Ex   

- ​	流水线CPU第三阶段
- ​	执行指令所需求的操作

![image-20230612104348124](https://gitee.com/guoweijing/photo/raw/master/202306121043181.png)

​	执行是指对获取的操作数进行指令所指定的算数或逻辑运算

```verilog
module Pipeline_EX( 
input[31:0] PC_in_EX, //PC输入
input[31:0] Rs1_in_EX, //操作数1输入
input[31:0] Rs2_in_EX, //操作数2输入
input[31:0] Imm_in_EX , //立即数输入
input ALUSrc_B_in_EX , //ALU B选择
input[31:0] ALU_control_in_EX, //ALU选择控制
output [31:0] PC_out_EX, //PC输出
output [31:0] PC4_out_EX, //PC+4输出
output zero_out_EX, //ALU判0输出
output [31:0] ALU_out_EX, //ALU计算输出
output [31:0] Rs2_out_EX, //操作数2输出
output [31:0] dMem_out_EX
);
```



#### 3.6 Pipeline—Ex_reg_Mem    

- ​	流水线CPU执行和访存之间的寄存器
- ​	存储ALU数据和控制信号

![image-20230612104525774](https://gitee.com/guoweijing/photo/raw/master/202306121045829.png)

​	寄存EX级的输出指令，分割EX级和MEM级的指令或控制信号，防止相互干扰，在EX级执行结束时将指令的控制信号传递至下一级.

```verilog
module Ex_reg_Mem( 
input [31:0]PC_cur_in_EXMem,
output reg [31:0]PC_cur_out_EXMem,
input valid_in_EXMem,
input clk_EXMem, //寄存器时钟
input rst_EXMem, //寄存器复位
input [31:0]dMem_out_EX,
output reg[31:0]dMem_out_EXMem,
input en_EXMem, //寄存器使能
input[31:0] PC_in_EXMem, //PC输入
input[31:0] PC4_in_EXMem, //PC+4输入
input [4:0] Rd_addr_EXMem, //写目的寄存器地址输入
input zero_in_EXMem, //zero
input[31:0] ALU_in_EXMem, //ALU输入
input[31:0] Rs2_in_EXMem ,//操作数2输入
input Branch_in_EXMem, //Branch
input MemRW_in_EXMem, //存储器读写
input [1:0]Jump_in_EXMem, //Jal、Jalr
input [2:0] MemtoReg_in_EXMem, //写回
input RegWrite_in_EXMem, //寄存器堆读写
input [31:0]Imm_in_EXMem,
input [31:0]inst_in_EXMem,
output reg valid_out_EXMem,
output reg[31:0] PC_out_EXMem, //PC输出
output reg[31:0] PC4_out_EXMem, //PC+4输出
output reg[4:0] Rd_addr_out_EXMem, //写目的寄存器输出
output reg zero_out_EXMem, //zero
output reg[31:0] ALU_out_EXMem, //ALU输出
output reg[31:0] Rs2_out_EXMem, //操作数2输出
output reg Branch_out_EXMem, //Branch
output reg MemRW_out_EXMem, //存储器读写
output reg[1:0] Jump_out_EXMem, //Jal、Jalr
output reg [2:0] MemtoReg_out_EXMem, //写回
output reg RegWrite_out_EXMem,//寄存器堆读写
output reg [31:0]Imm_out_EXMem,
output reg [31:0]inst_out_EXMem
);
```



#### 3.7 Pipeline—Mem   

- ​	流水线CPU第四阶段
- ​	访问存储器的操作

![image-20230612104709732](https://gitee.com/guoweijing/photo/raw/master/202306121047787.png)

​	存储器访问是指存储器访问指令将数据从存储器读出，或者写入存储器的过程

```verilog
module Pipeline_Mem( 
input zero_in_Mem, //zero
input Branch_in_Mem, //beq
input [1:0]Jump_in_Mem, //jal
output reg[1:0]PCSrc//PC选择控制输出
);
```



#### 3.8 Pipeline—Mem_reg_WB    

- ​	流水线CPU访存和写回之间的寄存器
- ​	存储ALU数据和存储器数据

![image-20230612104829715](https://gitee.com/guoweijing/photo/raw/master/202306121048777.png)

​	寄存Mem级的输出指令，以及输出数据，传递给写回阶段和寄存器堆。

```verilog
module Mem_reg_WB( 
input [31:0]PC_cur_in_MemWB,
output reg[31:0]PC_cur_out_MemWB,
input clk_MemWB, //寄存器时
input rst_MemWB, //寄存器复位
input en_MemWB, //寄存器使能
input valid_in_MemWB,
input[31:0] PC_in_MemWB,
input [31:0]inst_in_MemWB,
input[31:0] PC4_in_MemWB, //PC+4输入
input[4:0] Rd_addr_MemWB, //写目的地址输入
input[31:0] ALU_in_MemWB, //ALU输入
input[31:0] Dmem_data_MemWB, //存储器数据输入
input[2:0] MemtoReg_in_MemWB, //写回
input[31:0]Imm_in_MemWB,
input RegWrite_in_MemWB, //寄存器堆读写
output reg valid_out_MemWB,
output reg[31:0]inst_out_MemWB,
output reg[31:0] PC_out_MemWB,
output reg[31:0] PC4_out_MemWB, //PC+4输出
output reg[4:0] Rd_addr_out_MemWB, //写目的地址输出
output reg[31:0] ALU_out_MemWB, //ALU输出
output reg[31:0] DMem_data_out_MemWB,//存储器数据输出
output reg[2:0] MemtoReg_out_MemWB, //写回
output reg RegWrite_out_MemWB,//寄存器堆读写
output reg [31:0]Imm_out_MemWB,
input [1:0] PCSrc_in_MemWB,
output reg [1:0]PCSrc_out_MemWB
);
```



#### 3.9 Pipeline—WB   

- ​	流水线CPU第五阶段
- ​	结果写回寄存器堆的操作

​	![image-20230612104947986](https://gitee.com/guoweijing/photo/raw/master/202306121049056.png)

​	写回是指将指令执行的结果写回寄存器堆的过程；如果是普通运算指令，该结果值来源于‘执行’阶段计算的结果；如果是LOAD指令，该结果来源于‘访存’阶段从存储器读取出来的数据；如果是跳转指令，该结果来源于PC+4。

```verilog
module Pipeline_WB( 
input[31:0] PC4_in_WB, //PC+4输入
input[31:0] ALU_in_WB, //ALU结果输出
input[31:0] PC_in_WB,//新pc,auipc
input [31:0] Imm_in_WB,//立即数输出,lui
input[31:0] Dmem_data_WB, //存储器数据输入
input[2:0] MemtoReg_in_WB, //写回选择控制
output [31:0] Data_out_WB //写回数据输出
);
```



#### 3.10 CPU集成

​	拷贝下列模块到Pipeline_CPU工程目录：Pipeline_IF、IF_reg_ID、Pipeline_ID、ID_reg_Ex、Pipeline_Ex、Ex_reg_Mem、Pipeline_Mem、Mem_reg_WB、Pipeline_WB

![image-20230612105258396](https://gitee.com/guoweijing/photo/raw/master/202306121052473.png)

```verilog
module 	  Pipeline_CPU( 
	input  	clk,            	//时钟
	input  	rst,           	    //复位
	input[31:0]  	Data_in,    //存储器数据输入
	input[31:0]     inst_IF,    //取指阶段指令
	output [31:0]  	PC_out_IF,	//取指阶段PC输出
	output [31:0]  	PC_out_ID,	//译码阶段PC输出
	output [31:0]  	inst_ID,	//译码阶段指令
	output [31:0]  	PC_out_Ex,	//执行阶段PC输出
	output [31:0]  	MemRW_Ex,	//执行阶段存储器读写	
	output [31:0]  	MemRW_Mem,	//访存阶段存储器读写	
	output [31:0]  	Addr_out,	//地址输出
	output [31:0]  	Data_out,   //CPU数据输出
	output [31:0]  	Data_out_WB //写回数据输出
);  
```



## 实验5-2： IF,ID设计与集成

### 1.实验目的

1. ​	理解流水线CPU的基本原理和组织结构
2. ​	掌握五级流水线的工作过程和设计方法
3. ​	理解流水线取指、译码的设计原理
4. ​	设计流水线测试程序

### 2.实验任务

#### 任务一：设计取指（IF）、译码（ID）模块，替换实验九的流水线CPU并完成集成

- ​	设计取指模块，替换OExp05-1的取指模块并完成集成
- ​	设计译码模块，替换OExp05-1的译码模块并完成集成

#### 任务二：设计流水线测试方案并完成测试	

### 3.实验流程

#### 3.1取指模块 IF 设计

![image-20230612110723284](https://gitee.com/guoweijing/photo/raw/master/202306121107414.png)

```verilog
module Pipeline_IF( 
input [31:0]inst_in_IF,
output reg[31:0]inst_out_IF,
input clk_IF, //时钟
input rst_IF, //复位
input en_IF, //使能
input [31:0] PC_in_jalr,
input [31:0] PC4_in_IF,
input [31:0] PC_in_IF, //取指令PC输入
input [1:0]PCSrc, //PC输入选择
input NOP_IFID,
output  reg[31:0] PC_out_IF //PC输出
); 
wire [31:0] new_pc;
MUX4T1 MUX4T1_0(
.s(PCSrc),
.I0(PC4_in_IF),
.I1(PC_in_IF),
.I2(PC_in_jalr),
.I3(0),//PCSrc不会等于4的
.o(new_pc));
    always@(*) begin
    if(rst_IF==1)begin
     PC_out_IF=0;
     inst_out_IF=0;
     end
    else if (en_IF)begin
    if(NOP_IFID)
    begin
    PC_out_IF=new_pc-4;
    inst_out_IF=inst_in_IF;
    end
    else begin
    PC_out_IF=new_pc;
    inst_out_IF=inst_in_IF;
    end
    end
    end
endmodule
```



#### 3.2取指-译码寄存器设计

```verilog
module IF_Reg_ID( 
input clk_IFID, //寄存器时钟
input rst_IFID, //寄存器复位
input en_IFID, //寄存器使能
input [31:0] PC_in_IFID, //PC输入
input [31:0] inst_in_IFID, //指令输入
output reg [31:0] PC_out_IFID, //PC输出
output reg [31:0] inst_out_IFID, //指令输出
output reg valid_IFID//寄存器有效
);
    always@(posedge clk_IFID or posedge rst_IFID)
    begin
    if(rst_IFID==1)
    begin
    PC_out_IFID=0;
    inst_out_IFID=0;
    valid_IFID=0;
    end
    else//rst=0
    begin
    if(NOP_IFID)//NOP_IFID=1设置valid_IFID=0
    begin
    if(en_IFID==1)
    begin
    inst_out_IFID=32'h00000013;
    valid_IFID=0;  
    PC_out_IFID=PC_in_IFID;
    end
    else begin
    inst_out_IFID=32'h00000013;
    valid_IFID=0;
    end
    end
    else//NOP_IFID=0
    begin
    if(en_IFID)//NOP_IFID=0 and en_IFID=1
    begin
    valid_IFID=1;
    PC_out_IFID=PC_in_IFID;
    inst_out_IFID=inst_in_IFID;
    end
 //en_IFID==0就保持不变
    end
    end
    end
endmodule
```



#### 3.3 译码模块ID设计

![image-20230612111102150](https://gitee.com/guoweijing/photo/raw/master/202306121111273.png)

```verilog
module Pipeline_ID( 
input clk_ID, //时钟
input rst_ID, //复位
input RegWrite_in_ID, //寄存器堆使能
input [4:0] Rd_addr_ID, //写目的地址输入
input [31:0] Wt_data_ID, //写数据输入
input [31:0] Inst_in_ID, //指令输入
output [4:0] Rd_addr_out_ID, //写目的地址输出
output wire [31:0] Rs1_out_ID, //操作数1输出
output wire [31:0] Rs2_out_ID, //操作数2输出
output wire [31:0] Imm_out_ID, //立即数输出
output wire ALUSrc_B_ID, //ALU B端输入选择
output wire [31:0] ALU_control_ID,//ALU控制
output wire Branch_ID, //Branch控制
//output reg BranchN_ID, //Bne控制a
output wire MemRW_ID, //存储器读写
output wire [1:0] Jump_ID, //Jal,Jalr控制
output wire [2:0] MemtoReg_ID, //寄存器写回选择
output wire RegWrite_out_ID) ;//寄存器堆读写

wire [2:0]ImmSel;

assign Rd_addr_out_ID=Inst_in_ID[11:7];

RegFile Regs(
.clk(~clk_ID),
.rst(rst_ID),
.wen(RegWrite_in_ID),
.rs1(Inst_in_ID[19:15]),
.rs2(Inst_in_ID[24:20]),
.rd(Rd_addr_ID),
.i_data(Wt_data_ID),
.rs1_val(Rs1_out_ID),
.rs2_val(Rs2_out_ID));

ImmGen ImmGen_0(
.ImmSel(ImmSel),
.inst_field(Inst_in_ID),
.Imm_out(Imm_out_ID));


SCPU_ctrl SCPU_ctrl_0(
.inst_field(Inst_in_ID),
.OPcode(Inst_in_ID[6:0]),
.Fun3(Inst_in_ID[14:12]),
.Fun7(Inst_in_ID[30]),
.ImmSel(ImmSel),
.ALUSrc_B(ALUSrc_B_ID),
.MemtoReg(MemtoReg_ID),
.Jump(Jump_ID),
.Branch(Branch_ID),
.RegWrite(RegWrite_out_ID),
.MemRW(MemRW_ID),
.ALU_Control(ALU_control_ID));

endmodule

```



#### 3.4译码-执行寄存器设计

```verilog
module ID_Reg_EX(
input clk_IDEX,
input rst_IDEX,
input en_IDEX,
input NOP_IDEX,
input valid_in_IDEX,
input [31:0]PC_in_IDEX,
input [31:0]inst_in_IDEX,
//input [31:0]Wt_addr_IDEX,
input [31:0]Rs1_in_IDEX,
input [31:0]Rs2_in_IDEX,
input [31:0]Imm_in_IDEX,
input ALUSrc_B_in_IDEX,
input [31:0] ALU_control_in_IDEX,
input Branch_in_IDEX,
input MemRW_in_IDEX,
input [1:0]Jump_in_IDEX,
input [2:0]MemtoReg_in_IDEX,
input RegWrite_in_IDEX,
input [4:0] Rd_addr_IDEX,
output reg[31:0]inst_out_IDEX,
output reg[31:0]PC_out_IDEX,
//output reg[4:0]Wt_addr_out_IDEX,
output reg[31:0] Rs1_out_IDEX,
output reg[31:0] Rs2_out_IDEX,
output reg[31:0] Imm_out_IDEX,
output reg ALUSrc_B_IDEX,
output reg[31:0]ALU_control_out_IDEX,
output reg Branch_out_IDEX,
output reg[1:0] Jump_out_IDEX,
output reg MemRW_out_IDEX,
output reg RegWrite_out_IDEX,
output reg [2:0]MemtoReg_out_IDEX,
output reg [4:0] Rd_addr_out_IDEX,
output reg valid_out_IDEX
    );
    always@(posedge clk_IDEX or posedge rst_IDEX) begin
    if(rst_IDEX)
    begin  
    PC_out_IDEX=0;
     Rs1_out_IDEX=0;
    Rs2_out_IDEX=0;
    Imm_out_IDEX=0;
    ALUSrc_B_IDEX=0;
    ALU_control_out_IDEX=0;
    Branch_out_IDEX=0;
    Jump_out_IDEX=0;
    MemRW_out_IDEX=0;
    RegWrite_out_IDEX=0;
    MemtoReg_out_IDEX=0;
    Rd_addr_out_IDEX=0;
    inst_out_IDEX=0;
    valid_out_IDEX=0;
    end
    else 
    begin
    if(NOP_IDEX)
    begin
//    valid_out_IDEX=0;
//    inst_out_IDEX=32'h00000013;
//    valid_out_IDEX=0;
//disable the RegWrite and MemRW
   PC_out_IDEX=PC_in_IDEX;
    Rs1_out_IDEX=Rs1_in_IDEX;
    Rs2_out_IDEX=Rs2_in_IDEX;
    Imm_out_IDEX=Imm_in_IDEX;
    ALUSrc_B_IDEX=ALUSrc_B_in_IDEX;
    ALU_control_out_IDEX=ALU_control_in_IDEX;
    Branch_out_IDEX=Branch_in_IDEX;
    Jump_out_IDEX=Jump_in_IDEX;
    MemRW_out_IDEX=0;
    RegWrite_out_IDEX=0;
    MemtoReg_out_IDEX=MemtoReg_in_IDEX;
    Rd_addr_out_IDEX=Rd_addr_IDEX;
    inst_out_IDEX=inst_in_IDEX;
    valid_out_IDEX=0;
    end
    else if(en_IDEX)
    begin
    PC_out_IDEX=PC_in_IDEX;
    Rs1_out_IDEX=Rs1_in_IDEX;
    Rs2_out_IDEX=Rs2_in_IDEX;
    Imm_out_IDEX=Imm_in_IDEX;
    ALUSrc_B_IDEX=ALUSrc_B_in_IDEX;
    ALU_control_out_IDEX=ALU_control_in_IDEX;
    Branch_out_IDEX=Branch_in_IDEX;
    Jump_out_IDEX=Jump_in_IDEX;
    MemRW_out_IDEX=MemRW_in_IDEX;
    RegWrite_out_IDEX=RegWrite_in_IDEX;
    MemtoReg_out_IDEX=MemtoReg_in_IDEX;
    Rd_addr_out_IDEX=Rd_addr_IDEX;
    inst_out_IDEX=inst_in_IDEX;
    valid_out_IDEX=valid_in_IDEX;
    end
    end
    end
endmodule

```



#### 3.5 IF,ID模块替换与CPU集成

1. ​	移除工程中的取指、译码模块

2. ​	建议用Exp05-1资源重建工程

![image-20230612111358382](https://gitee.com/guoweijing/photo/raw/master/202306121113473.png)



## 实验5-3：Ex、Mem、WB设计与集成

### 1.实验目的

1. ​	理解流水线CPU的基本原理和组织结构
2. ​	掌握五级流水线的工作过程和设计方法
3. ​	理解流水线执行、存储器访问、写回的原理
4. ​	设计流水线测试程序

### 2.实验任务

#### 任务一：设计执行(Ex)存储器访问(Mem)写回(WB)模块，替换lab04的流水线CPU并完成集成

- 设计执行模块，替换OExp05-2的执行模块并完成集成
- 设计访存模块，替换OExp05-2的访存模块并完成集成
- 设计写回模块，替换OExp05-2的写回模块并完成集成

#### 任务二：设计流水线测试方案并完成测试	

### 3.实验流程

#### 3.1执行模块Ex设计

![image-20230612132500156](https://gitee.com/guoweijing/photo/raw/master/202306121325342.png)

```verilog
module Pipeline_EX( 
input[31:0] PC_in_EX, //PC输入
input[31:0] Rs1_in_EX, //操作数1输入
input[31:0] Rs2_in_EX, //操作数2输入
input[31:0] Imm_in_EX , //立即数输入
input ALUSrc_B_in_EX , //ALU B选择
input[31:0] ALU_control_in_EX, //ALU选择控制
output [31:0] PC_out_EX, //PC输出
output [31:0] PC4_out_EX, //PC+4输出
output zero_out_EX, //ALU判0输出
output [31:0] ALU_out_EX, //ALU计算输出
output [31:0] Rs2_out_EX, //操作数2输出
output [31:0] dMem_out_EX
);

assign Rs2_out_EX=Rs2_in_EX;
wire null;

wire [31:0] ALU_nextdata;
add32 add_32_1(
.a(PC_in_EX),
.b(Imm_in_EX),
.c(PC_out_EX));

add32 add_32_2(
.a(PC_in_EX),
.b(4),
.c(PC4_out_EX));

MUX2T1 MUX2T1_32_1(
.I0(Rs2_in_EX),
.I1(Imm_in_EX),
.s(ALUSrc_B_in_EX),
.o(ALU_nextdata));

ALU ALU_0(
.a_val(Rs1_in_EX),
.b_val(ALU_nextdata),
.ctrl(ALU_control_in_EX),
.result(ALU_out_EX),
.zero(zero_out_EX));

ALU ALU_1(
.a_val(Rs1_in_EX),
.b_val(ALU_nextdata),
.ctrl(ALU_control_in_EX),
.result(dMem_out_EX),
.zero(null));


endmodule
```



#### 3.2执行-访存寄存器设计

```verilog
module Ex_reg_Mem( 
input [31:0]PC_cur_in_EXMem,
output reg [31:0]PC_cur_out_EXMem,
input valid_in_EXMem,
input clk_EXMem, //寄存器时钟
input rst_EXMem, //寄存器复位
input [31:0]dMem_out_EX,
output reg[31:0]dMem_out_EXMem,
input en_EXMem, //寄存器使能
input[31:0] PC_in_EXMem, //PC输入
input[31:0] PC4_in_EXMem, //PC+4输入
input [4:0] Rd_addr_EXMem, //写目的寄存器地址输入
input zero_in_EXMem, //zero
input[31:0] ALU_in_EXMem, //ALU输入
input[31:0] Rs2_in_EXMem ,//操作数2输入
input Branch_in_EXMem, //Branch
input MemRW_in_EXMem, //存储器读写
input [1:0]Jump_in_EXMem, //Jal、Jalr
input [2:0] MemtoReg_in_EXMem, //写回
input RegWrite_in_EXMem, //寄存器堆读写
input [31:0]Imm_in_EXMem,
input [31:0]inst_in_EXMem,
output reg valid_out_EXMem,
output reg[31:0] PC_out_EXMem, //PC输出
output reg[31:0] PC4_out_EXMem, //PC+4输出
output reg[4:0] Rd_addr_out_EXMem, //写目的寄存器输出
output reg zero_out_EXMem, //zero
output reg[31:0] ALU_out_EXMem, //ALU输出
output reg[31:0] Rs2_out_EXMem, //操作数2输出
output reg Branch_out_EXMem, //Branch
output reg MemRW_out_EXMem, //存储器读写
output reg[1:0] Jump_out_EXMem, //Jal、Jalr
output reg [2:0] MemtoReg_out_EXMem, //写回
output reg RegWrite_out_EXMem,//寄存器堆读写
output reg [31:0]Imm_out_EXMem,
output reg [31:0]inst_out_EXMem
);
always@(posedge clk_EXMem or posedge rst_EXMem)begin
if(rst_EXMem) begin
PC_out_EXMem=0;
PC4_out_EXMem=0;
Rd_addr_out_EXMem=0;
zero_out_EXMem=0;
ALU_out_EXMem=0;
Rs2_out_EXMem=0;
Branch_out_EXMem=0;
MemRW_out_EXMem=0;
Jump_out_EXMem=0;
MemtoReg_out_EXMem=0;
RegWrite_out_EXMem=0;
Imm_out_EXMem=0;
inst_out_EXMem=0;
valid_out_EXMem=0;
PC_cur_out_EXMem=0;
dMem_out_EXMem=0;
end
else begin
if(en_EXMem)begin
PC_out_EXMem=PC_in_EXMem;
PC4_out_EXMem=PC4_in_EXMem;
Rd_addr_out_EXMem=Rd_addr_EXMem;
zero_out_EXMem=zero_in_EXMem;
ALU_out_EXMem=ALU_in_EXMem;
Rs2_out_EXMem=Rs2_in_EXMem;
Branch_out_EXMem=Branch_in_EXMem;
MemRW_out_EXMem=MemRW_in_EXMem;
Jump_out_EXMem=Jump_in_EXMem;
MemtoReg_out_EXMem=MemtoReg_in_EXMem;
RegWrite_out_EXMem=RegWrite_in_EXMem;
Imm_out_EXMem=Imm_in_EXMem;
inst_out_EXMem=inst_in_EXMem;
valid_out_EXMem=valid_in_EXMem;
dMem_out_EXMem=dMem_out_EX;
PC_cur_out_EXMem=PC_cur_in_EXMem;
end
end
end
endmodule

```



#### 3.3访存模块MEM设计

![image-20230612132658469](https://gitee.com/guoweijing/photo/raw/master/202306121326566.png)

```verilog
module Pipeline_Mem( 
input zero_in_Mem, //zero
input Branch_in_Mem, //beq
input [1:0]Jump_in_Mem, //jal
output reg[1:0]PCSrc//PC选择控制输出
);

wire isBranch;
always@*begin 
if(isBranch|Jump_in_Mem[0])
PCSrc=2'b01;
else if(Jump_in_Mem[1])
PCSrc=2'b10;
else
PCSrc=2'b00;
end

and_2 and_2_1(
.Op1(Branch_in_Mem),
.Op2(zero_in_Mem),
.Res(isBranch));



endmodule
```



#### 3.4访存-写回寄存器设计

```verilog
module Mem_reg_WB( 
input [31:0]PC_cur_in_MemWB,
output reg[31:0]PC_cur_out_MemWB,
input clk_MemWB, //寄存器时
input rst_MemWB, //寄存器复位
input en_MemWB, //寄存器使能
input valid_in_MemWB,
input[31:0] PC_in_MemWB,
input [31:0]inst_in_MemWB,
input[31:0] PC4_in_MemWB, //PC+4输入
input[4:0] Rd_addr_MemWB, //写目的地址输入
input[31:0] ALU_in_MemWB, //ALU输入
input[31:0] Dmem_data_MemWB, //存储器数据输入
input[2:0] MemtoReg_in_MemWB, //写回
input[31:0]Imm_in_MemWB,
input RegWrite_in_MemWB, //寄存器堆读写
output reg valid_out_MemWB,
output reg[31:0]inst_out_MemWB,
output reg[31:0] PC_out_MemWB,
output reg[31:0] PC4_out_MemWB, //PC+4输出
output reg[4:0] Rd_addr_out_MemWB, //写目的地址输出
output reg[31:0] ALU_out_MemWB, //ALU输出
output reg[31:0] DMem_data_out_MemWB,//存储器数据输出
output reg[2:0] MemtoReg_out_MemWB, //写回
output reg RegWrite_out_MemWB,//寄存器堆读写
output reg [31:0]Imm_out_MemWB,
input [1:0] PCSrc_in_MemWB,
output reg [1:0]PCSrc_out_MemWB
);
always@(posedge clk_MemWB or posedge rst_MemWB)begin
if(rst_MemWB)
begin
PC4_out_MemWB=0;
Rd_addr_out_MemWB=0;
ALU_out_MemWB=0;
DMem_data_out_MemWB=0;
MemtoReg_out_MemWB=0;
RegWrite_out_MemWB=0;
Imm_out_MemWB=0;
PC_out_MemWB=0;
inst_out_MemWB=0;
valid_out_MemWB=0;
PC_cur_out_MemWB=0;
PCSrc_out_MemWB=0;
end
else 
begin
if(en_MemWB)begin
PC4_out_MemWB=PC4_in_MemWB;
Rd_addr_out_MemWB=Rd_addr_MemWB;
ALU_out_MemWB=ALU_in_MemWB;
DMem_data_out_MemWB=Dmem_data_MemWB;
MemtoReg_out_MemWB=MemtoReg_in_MemWB;
RegWrite_out_MemWB=RegWrite_in_MemWB;
Imm_out_MemWB=Imm_in_MemWB;
PC_out_MemWB=PC_in_MemWB;
valid_out_MemWB=1;
inst_out_MemWB=inst_in_MemWB;
PC_cur_out_MemWB=PC_cur_in_MemWB;
PCSrc_out_MemWB=PCSrc_in_MemWB;
end
end
end
endmodule
```



#### 3.5写回模块WB设计

![image-20230612133015964](https://gitee.com/guoweijing/photo/raw/master/202306121330037.png)

```verilog
module Pipeline_WB( 
input[31:0] PC4_in_WB, //PC+4输入
input[31:0] ALU_in_WB, //ALU结果输出
input[31:0] PC_in_WB,//新pc,auipc
input [31:0] Imm_in_WB,//立即数输出,lui
input[31:0] Dmem_data_WB, //存储器数据输入
input[2:0] MemtoReg_in_WB, //写回选择控制
output [31:0] Data_out_WB //写回数据输出
);
    MUX5T1 MUX5T1_32_0(
    .s(MemtoReg_in_WB),
    .I0(ALU_in_WB),
    .I1(Dmem_data_WB),
    .I2(PC4_in_WB),
    .I3(Imm_in_WB),
    .I4(PC_in_WB),
    .o(Data_out_WB));   
endmodule
```



#### 3.6 Ex、Mem、WB模块替换与流水线CPU集成

- ​	移除工程中的执行、访存、写回模块

- ​	建议用Exp05-2资源重建工程

![image-20230612133131543](https://gitee.com/guoweijing/photo/raw/master/202306121331660.png)



#### 3.7物理验证

![image-20230612150916822](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230612150916822.png)

![30](C:\Users\86184\Desktop\30.JPG)

![34](C:\Users\86184\Desktop\34.JPG)



## 实验5-4：流水线处理器—冒险与stall

### 1.实验目的

- 理解流水线CPU的基本原理和组织结构
- 掌握五级流水线的工作过程和设计方法
- 理解流水线CPU停机的原理与解决办法
- 设计流水线测试程序

### 2.实验任务

#### 任务一：集成设计利用stall解决冒险的流水线CPU，在lab05-3的基础上完成

- 设计冒险检测及stall消除冒险的流水线CPU
- 替换 lab05-3的CPU为本实验集成的带stall处理的流水线CPU

#### 任务二：设计流水线测试方案并完成测试

### 3.实验流程

#### 3.1 Stall 冒险检测及处理单元

​	根据冒险相关性特点停顿流水线

![image-20230612133645701](https://gitee.com/guoweijing/photo/raw/master/202306121336781.png)

- ​	检测数据冒险和控制冒险

- ​	根据冒险类型，停顿流水线

  

  ![image-20230612135135764](https://gitee.com/guoweijing/photo/raw/master/202306121351904.png)

  

```verilog
module stall( 
input rst_stall, //复位
input RegWrite_out_IDEX, //执行阶段寄存器写控制
input [4:0]Rd_addr_out_IDEX, //执行阶段寄存器写地址
input RegWrite_out_EXMem, //访存阶段寄存器写控制
input [4:0]Rd_addr_out_EXMem, //访存阶段寄存器写地址
input [4:0]Rs1_addr_ID, //译码阶段寄存器读地址1
input [4:0]Rs2_addr_ID, //译码阶段寄存器读地址2
input Rs1_used, //Rs1被使用
input Rs2_used, //Rs2被使用
input Branch_ID, //译码阶段b-type
input [1:0]Jump_ID, //译码阶段jal
input Branch_out_IDEX, //执行阶段b-type
input [1:0]Jump_out_IDEX, //执行阶段jal
input [1:0]PCSrc_out_MemWB,
input Branch_out_EXMem, //访存阶段b-type
input [1:0]Jump_out_EXMem, //访存阶段jal
input [4:0]Rs1_addr_IDEX,
input [4:0]Rs2_addr_IDEX,
output reg en_IF, //流水线寄存器的使能及NOP信号
output reg en_IFID,
output reg NOP_IFID,
output reg NOP_IDEx
); 
always@* begin
if(rst_stall)begin
//按理来说rst之后是要能正常用的
en_IF=1;
en_IFID=1;
NOP_IFID=0;
NOP_IDEx=0;
end
else begin
//Data_hazard
//if(RegWrite_out_EXMem&&Rs1_used&&Rs1_addr_ID!=0&&Rd_addr_out_EXMem==Rs1_addr_ID)
NOP_IFID=0;
NOP_IDEx=0;
en_IF=1;
en_IFID=1;
if((Rs1_used&&Rs1_addr_ID!=0&&Rd_addr_out_EXMem==Rs1_addr_ID)||(Rs1_used&&Rs1_addr_ID!=0&&Rd_addr_out_IDEX==Rs1_addr_ID)||(Rs2_used&&Rs2_addr_ID!=0&&Rd_addr_out_IDEX==Rs2_addr_ID)||(Rs2_used&&Rs2_addr_ID!=0&&Rd_addr_out_EXMem==Rs2_addr_ID)||(Rs2_used&&Rs2_addr_IDEX!=0&&Rd_addr_out_EXMem==Rs2_addr_IDEX)||(Rs1_used&&Rs1_addr_IDEX!=0&&Rd_addr_out_EXMem==Rs1_addr_IDEX))
begin
//插入stall
NOP_IDEx=1;
en_IF=0;
en_IFID=0;
end

//Control hazard||(Branch_out_EXMem==1||Jump_out_EXMem==2'b01||Jump_out_EXMem==2'b10)
else if((Branch_ID==1||Jump_ID==2'b01||Jump_ID==2'b10))
begin
NOP_IFID=1;
en_IF=0;
en_IFID=0;
end
else if((Branch_out_IDEX==1||Jump_out_IDEX==2'b01||Jump_out_IDEX==2'b10))
begin
NOP_IFID=1;
//NOP_IDEx=1;
en_IF=0;
en_IFID=0;
end
else if(Branch_out_EXMem==1||Jump_out_EXMem==2'b01||Jump_out_EXMem==2'b10)
begin
NOP_IFID=1;
//NOP_IDEx=1;
en_IF=0;
en_IFID=0;
end
else if(PCSrc_out_MemWB!=0)
begin
NOP_IFID=1;
en_IF=1;
en_IFID=1;
end
end
end
endmodule
```

#### 3.2 译码模块、流水线寄存器重新设计集成

##### 3.2.1取指-译码寄存器：IF_reg_ID修改

```verilog
module IF_Reg_ID( 
input clk_IFID, //寄存器时钟
input rst_IFID, //寄存器复位
input en_IFID, //寄存器使能
input [31:0] PC_in_IFID, //PC输入
input [31:0] inst_in_IFID, //指令输入
input NOP_IFID, //插入NOP使能
output reg [31:0] PC_out_IFID, //PC输出
output reg [31:0] inst_out_IFID, //指令输出
output reg valid_IFID//寄存器有效
);
    
    if(NOP_IFID)//NOP_IFID=1设置valid_IFID=0
    begin
    if(en_IFID==1)
    begin
    inst_out_IFID=32'h00000013;
    valid_IFID=0;  
    PC_out_IFID=PC_in_IFID;
    end
    else begin
    inst_out_IFID=32'h00000013;
    valid_IFID=0;
    end
```



##### 3.2.2译码模块Pipeline—ID修改

```verilog
module Pipeline_ID( 
`VGA_DBG_RegFile_Outputs
input clk_ID, //时钟
input rst_ID, //复位
input RegWrite_in_ID, //寄存器堆使能
input [4:0] Rd_addr_ID, //写目的地址输入
input [31:0] Wt_data_ID, //写数据输入
input [31:0] Inst_in_ID, //指令输入
output [4:0] Rd_addr_out_ID, //写目的地址输出
output wire [31:0] Rs1_out_ID, //操作数1输出
output wire [31:0] Rs2_out_ID, //操作数2输出
output wire [31:0] Imm_out_ID, //立即数输出
output wire ALUSrc_B_ID, //ALU B端输入选择
output wire [31:0] ALU_control_ID,//ALU控制
output wire Branch_ID, //Branch控制
//output reg BranchN_ID, //Bne控制a
output wire MemRW_ID, //存储器读写
output wire [1:0] Jump_ID, //Jal,Jalr控制
output wire [2:0] MemtoReg_ID, //寄存器写回选择
output wire RegWrite_out_ID) ;//寄存器堆读写
    
    RegFile Regs(
`VGA_DBG_RegFile_Arguments
.clk(~clk_ID),
.rst(rst_ID),
.wen(RegWrite_in_ID),
.rs1(Inst_in_ID[19:15]),
.rs2(Inst_in_ID[24:20]),
.rd(Rd_addr_ID),
.i_data(Wt_data_ID),
.rs1_val(Rs1_out_ID),
.rs2_val(Rs2_out_ID));

ImmGen ImmGen_0(
.ImmSel(ImmSel),
.inst_field(Inst_in_ID),
.Imm_out(Imm_out_ID));


SCPU_ctrl SCPU_ctrl_0(
.inst_field(Inst_in_ID),
.OPcode(Inst_in_ID[6:0]),
.Fun3(Inst_in_ID[14:12]),
.Fun7(Inst_in_ID[30]),
.ImmSel(ImmSel),
.ALUSrc_B(ALUSrc_B_ID),
.MemtoReg(MemtoReg_ID),
.Jump(Jump_ID),
.Branch(Branch_ID),
.RegWrite(RegWrite_out_ID),
.MemRW(MemRW_ID),
.ALU_Control(ALU_control_ID));
```



##### 3.2.3译码-执行寄存器：ID_reg_EX修改

```verilog
module ID_Reg_EX(
input clk_IDEX,
input rst_IDEX,
input en_IDEX,
input NOP_IDEX,
input valid_in_IDEX,
input [31:0]PC_in_IDEX,
input [31:0]inst_in_IDEX,
//input [31:0]Wt_addr_IDEX,
input [31:0]Rs1_in_IDEX,
input [31:0]Rs2_in_IDEX,
input [31:0]Imm_in_IDEX,
input ALUSrc_B_in_IDEX,
input [31:0] ALU_control_in_IDEX,
input Branch_in_IDEX,
input MemRW_in_IDEX,
input [1:0]Jump_in_IDEX,
input [2:0]MemtoReg_in_IDEX,
input RegWrite_in_IDEX,
input [4:0] Rd_addr_IDEX,
output reg[31:0]inst_out_IDEX,
output reg[31:0]PC_out_IDEX,
//output reg[4:0]Wt_addr_out_IDEX,
output reg[31:0] Rs1_out_IDEX,
output reg[31:0] Rs2_out_IDEX,
output reg[31:0] Imm_out_IDEX,
output reg ALUSrc_B_IDEX,
output reg[31:0]ALU_control_out_IDEX,
output reg Branch_out_IDEX,
output reg[1:0] Jump_out_IDEX,
output reg MemRW_out_IDEX,
output reg RegWrite_out_IDEX,
output reg [2:0]MemtoReg_out_IDEX,
output reg [4:0] Rd_addr_out_IDEX,
output reg valid_out_IDEX
    );
    
    if(NOP_IDEX)
    begin
//disable the RegWrite and MemRW
   PC_out_IDEX=PC_in_IDEX;
    Rs1_out_IDEX=Rs1_in_IDEX;
    Rs2_out_IDEX=Rs2_in_IDEX;
    Imm_out_IDEX=Imm_in_IDEX;
    ALUSrc_B_IDEX=ALUSrc_B_in_IDEX;
    ALU_control_out_IDEX=ALU_control_in_IDEX;
    Branch_out_IDEX=Branch_in_IDEX;
    Jump_out_IDEX=Jump_in_IDEX;
    MemRW_out_IDEX=0;
    RegWrite_out_IDEX=0;
    MemtoReg_out_IDEX=MemtoReg_in_IDEX;
    Rd_addr_out_IDEX=Rd_addr_IDEX;
    inst_out_IDEX=inst_in_IDEX;
    valid_out_IDEX=0;
    end
```



##### 3.2.4执行-访存寄存器：Ex_reg_Mem修改

```verilog
module Ex_reg_Mem( 
input [31:0]PC_cur_in_EXMem,
output reg [31:0]PC_cur_out_EXMem,
input valid_in_EXMem,
input clk_EXMem, //寄存器时钟
input rst_EXMem, //寄存器复位
input [31:0]dMem_out_EX,
output reg[31:0]dMem_out_EXMem,
input en_EXMem, //寄存器使能
input[31:0] PC_in_EXMem, //PC输入
input[31:0] PC4_in_EXMem, //PC+4输入
input [4:0] Rd_addr_EXMem, //写目的寄存器地址输入
input zero_in_EXMem, //zero
input[31:0] ALU_in_EXMem, //ALU输入
input[31:0] Rs2_in_EXMem ,//操作数2输入
input Branch_in_EXMem, //Branch
input MemRW_in_EXMem, //存储器读写
input [1:0]Jump_in_EXMem, //Jal、Jalr
input [2:0] MemtoReg_in_EXMem, //写回
input RegWrite_in_EXMem, //寄存器堆读写
input [31:0]Imm_in_EXMem,
input [31:0]inst_in_EXMem,
output reg valid_out_EXMem,
output reg[31:0] PC_out_EXMem, //PC输出
output reg[31:0] PC4_out_EXMem, //PC+4输出
output reg[4:0] Rd_addr_out_EXMem, //写目的寄存器输出
output reg zero_out_EXMem, //zero
output reg[31:0] ALU_out_EXMem, //ALU输出
output reg[31:0] Rs2_out_EXMem, //操作数2输出
output reg Branch_out_EXMem, //Branch
output reg MemRW_out_EXMem, //存储器读写
output reg[1:0] Jump_out_EXMem, //Jal、Jalr
output reg [2:0] MemtoReg_out_EXMem, //写回
output reg RegWrite_out_EXMem,//寄存器堆读写
output reg [31:0]Imm_out_EXMem,
output reg [31:0]inst_out_EXMem
);
always@(posedge clk_EXMem or posedge rst_EXMem)begin
if(rst_EXMem) begin
PC_out_EXMem=0;
PC4_out_EXMem=0;
Rd_addr_out_EXMem=0;
zero_out_EXMem=0;
ALU_out_EXMem=0;
Rs2_out_EXMem=0;
Branch_out_EXMem=0;
MemRW_out_EXMem=0;
Jump_out_EXMem=0;
MemtoReg_out_EXMem=0;
RegWrite_out_EXMem=0;
Imm_out_EXMem=0;
inst_out_EXMem=0;
valid_out_EXMem=0;
PC_cur_out_EXMem=0;
dMem_out_EXMem=0;
end
else begin
if(en_EXMem)begin
PC_out_EXMem=PC_in_EXMem;
PC4_out_EXMem=PC4_in_EXMem;
Rd_addr_out_EXMem=Rd_addr_EXMem;
zero_out_EXMem=zero_in_EXMem;
ALU_out_EXMem=ALU_in_EXMem;
Rs2_out_EXMem=Rs2_in_EXMem;
Branch_out_EXMem=Branch_in_EXMem;
MemRW_out_EXMem=MemRW_in_EXMem;
Jump_out_EXMem=Jump_in_EXMem;
MemtoReg_out_EXMem=MemtoReg_in_EXMem;
RegWrite_out_EXMem=RegWrite_in_EXMem;
Imm_out_EXMem=Imm_in_EXMem;
inst_out_EXMem=inst_in_EXMem;
valid_out_EXMem=valid_in_EXMem;
dMem_out_EXMem=dMem_out_EX;
PC_cur_out_EXMem=PC_cur_in_EXMem;
end
end
end
endmodule

```



##### 3.2.5访存-写回寄存器：Mem_reg_WB修改

```verilog
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 2022/05/09 22:02:37
// Design Name: 
// Module Name: Mem_Reg_WB
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module Mem_reg_WB( 
input [31:0]PC_cur_in_MemWB,
output reg[31:0]PC_cur_out_MemWB,
input clk_MemWB, //寄存器时
input rst_MemWB, //寄存器复位
input en_MemWB, //寄存器使能
input valid_in_MemWB,
input[31:0] PC_in_MemWB,
input [31:0]inst_in_MemWB,
input[31:0] PC4_in_MemWB, //PC+4输入
input[4:0] Rd_addr_MemWB, //写目的地址输入
input[31:0] ALU_in_MemWB, //ALU输入
input[31:0] Dmem_data_MemWB, //存储器数据输入
input[2:0] MemtoReg_in_MemWB, //写回
input[31:0]Imm_in_MemWB,
input RegWrite_in_MemWB, //寄存器堆读写
output reg valid_out_MemWB,
output reg[31:0]inst_out_MemWB,
output reg[31:0] PC_out_MemWB,
output reg[31:0] PC4_out_MemWB, //PC+4输出
output reg[4:0] Rd_addr_out_MemWB, //写目的地址输出
output reg[31:0] ALU_out_MemWB, //ALU输出
output reg[31:0] DMem_data_out_MemWB,//存储器数据输出
output reg[2:0] MemtoReg_out_MemWB, //写回
output reg RegWrite_out_MemWB,//寄存器堆读写
output reg [31:0]Imm_out_MemWB,
input [1:0] PCSrc_in_MemWB,
output reg [1:0]PCSrc_out_MemWB
);
always@(posedge clk_MemWB or posedge rst_MemWB)begin
if(rst_MemWB)
begin
PC4_out_MemWB=0;
Rd_addr_out_MemWB=0;
ALU_out_MemWB=0;
DMem_data_out_MemWB=0;
MemtoReg_out_MemWB=0;
RegWrite_out_MemWB=0;
Imm_out_MemWB=0;
PC_out_MemWB=0;
inst_out_MemWB=0;
valid_out_MemWB=0;
PC_cur_out_MemWB=0;
PCSrc_out_MemWB=0;
end
else 
begin
if(en_MemWB)begin
PC4_out_MemWB=PC4_in_MemWB;
Rd_addr_out_MemWB=Rd_addr_MemWB;
ALU_out_MemWB=ALU_in_MemWB;
DMem_data_out_MemWB=Dmem_data_MemWB;
MemtoReg_out_MemWB=MemtoReg_in_MemWB;
RegWrite_out_MemWB=RegWrite_in_MemWB;
Imm_out_MemWB=Imm_in_MemWB;
PC_out_MemWB=PC_in_MemWB;
valid_out_MemWB=1;
inst_out_MemWB=inst_in_MemWB;
PC_cur_out_MemWB=PC_cur_in_MemWB;
PCSrc_out_MemWB=PCSrc_in_MemWB;
end
end
end
endmodule

```

#### 3.3 物理验证

##### 数据冒险：

![image-20230612144144069](https://gitee.com/guoweijing/photo/raw/master/202306121441171.png)

![20](C:\Users\86184\Desktop\20.JPG)

**这里我们可以看到，由于pc=20指令x7依赖于pc=1c指令，因此需要等到1c处指令写回WB时，20处指令才可以译码ID。**

![image-20230612144945727](https://gitee.com/guoweijing/photo/raw/master/202306121449816.png)

![18](C:\Users\86184\Desktop\18.JPG)

**同样，这里pc=18的指令的x5依赖于pc=14指令，需等其写回再进行译码，插入stall**



##### 控制冒险：

![image-20230612144422915](https://gitee.com/guoweijing/photo/raw/master/202306121444010.png)

![38](C:\Users\86184\Desktop\38.JPG)

**当pc=38,beq跳转，这是需要插入stall，等到其计算并将pc写回，在进行下一条指令，可以看到，跳转到了pc=44处.**

![image-20230612144523139](https://gitee.com/guoweijing/photo/raw/master/202306121445232.png)

![68](C:\Users\86184\Desktop\68.JPG)

**我们可以看到当pc=68时，执行跳转指令，中间插入stall，直到跳转指令执行完毕，pc变成70.**