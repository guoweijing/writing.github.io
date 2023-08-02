---
title: 单周期CPU设计
categories:
- architecture
tag :

---       																							

# 实验四：单周期CPU设计

[TOC]

## 实验4-0：集成CPU核

### 一.实验目的

1.	**复习寄存器传输控制技术**
2.	**掌握CPU的核心组成：数据通路与控制器**
3. **设计数据通路的功能部件**
4. **进一步了解计算机系统的基本结构**
5. **熟练掌握IP核的使用方法**

### 二.实验任务

**利用数据通路和控制器两个IP核集成设计CPU（根据原理图采用RTL代码方式）**

![image-20230529103957916](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529103957916.png)

### 三.实验流程

#### 3.1分解CPU为两个IP核

​	**用二个IP核构建CPU**
​	**顶层工程名称延用OExp04-IP2CPU**
​	**顶层模块名：SCPU.v**

![image-20230529104144849](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529104144849.png)

#### 3.2添加二个核的封装文件到当前工程IP库目录

![image-20230529104259252](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529104259252.png)

#### 3.3利用verilog描述，在SCPU.v顶层进行模块调用和连接

```verilog
module SCPU_int(
input wire clk,
input wire rst,
input wire [31:0] inst_in,
input wire [31:0] Data_in,
input wire MIO_ready,
input wire INT0,
output wire CPU_MIO,
output wire MemRW,
output wire [31:0] Data_out,
output wire [31:0] Addr_out,
output wire [31:0] PC_out,
)
```

![image-20230529104631835](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529104631835.png)





## 实验4-1：CPU设计-数据通路DataPath

### 一.实验目的：

1.	**运用寄存器传输控制技术**
2.	**掌握CPU的核心：数据通路组成与原理**
3. **设计数据通路**
4. **学习测试方案的设计**
5. **学习测试程序的设计**

### 二.实验任务：

#### 任务一：设计实现数据通路（采用RTL实现）

- **ALU和Regs调用Exp01设计的模块（可直接加RTL）**
- **PC寄存器设计及PC通路建立**
- **ImmGen立即数生成模块设计**
- **此实验在Exp4-0的基础上完成，替换Exp4-0的数据通路核**

#### 任务二：设计数据通路测试方案并完成测试

- **通路测试：I-格式通路、R-格式通路**	

- **部件测试：ALU、Register Files**

  ![image-20230529105418819](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529105418819.png)

### 三.实验流程

#### 3.1添加一系列模块到IP目录

**and32、or32、ADC32、xor32、nor32、srl32、SignalExt_32、mux8to1_32、or_bit_32**
**add_32、 mux2to1_32 、 mux2to1_5、ALU、Regs、Ext_32、REG32、ImmGen**

#### 3.2 PC寄存器设计及PC通路建立

```verilog
module REG32(
input clk,
input rst,
input CE,
input [31:0]D,
output reg [31:0]Q
    );
    always @(*)
    begin
        if( rst == 1)
        begin 
             Q<=32'b0;
        end
        else begin
        Q<=D;
        end  
end
endmodule

```

#### 3.3 根据原理图设计数据通路顶层

![image-20230529105845339](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529105845339.png)

```verilog
module Datapath(
input wire clk,
input wire rst,
input wire[31:0] inst_field,
input wire[31:0] Data_in,
input wire[3:0] ALU_Control,
input wire[2:0] ImmSel,
input wire[1:0] MemtoReg,
input wire ALUSrc_B,
input wire [1:0]Jump,
input wire Branch,
input wire BranchN,
input wire RegWrite,
input wire INT0,
input wire mret,
input wire ill_instr,
input wire ecall,
output wire[31:0] PC_out,
output wire[31:0] Data_out,
output wire[31:0] ALU_out,
output wire [31:0] x0,
output wire [31:0] ra,
output wire [31:0] sp,
output wire [31:0] gp,
output wire [31:0] tp,
output wire [31:0] t0,
output wire [31:0] t1,
output wire [31:0] t2,
output wire [31:0] s0,
output wire [31:0] s1,
output wire [31:0] a0,
output wire [31:0] a1,
output wire [31:0] a2,
output wire [31:0] a3,
output wire [31:0] a4,
output wire [31:0] a5,
output wire [31:0] a6,
output wire [31:0] a7,
output wire [31:0] s2,
output wire [31:0] s3,
output wire [31:0] s4,
output wire [31:0] s5,
output wire [31:0] s6,
output wire [31:0] s7, 
output wire [31:0] s8,
output wire [31:0] s9,
output wire [31:0] s10,
output wire [31:0] s11,
output wire [31:0] t3,
output wire [31:0] t4,
output wire [31:0] t5,
output wire [31:0] t6
);
wire [31:0]Q;
wire [31:0]Imm_out;
wire [31:0] I1;
wire [31:0] I0;
wire ALU_zero;
wire [31:0] o0;
wire [31:0] Wt_data;
wire [31:0] ALU_res;
wire [31:0] pc_next;
wire [31:0] Rs1_data;
wire [31:0] Rs2_data;
wire [31:0] B;
wire [31:0] c0;
wire [31:0] c1;
wire [31:0] o1;
wire [31:0] pc;

ImmGen_0 ImmGen (
  .ImmSel(ImmSel),          // input wire [1 : 0] ImmSel
  .inst_field(inst_field),  // input wire [31 : 0] inst_field
  .Imm_out(Imm_out)        // output wire [31 : 0] Imm_out
);
MUX4T1_32_0 Mux4t0 (
  .I0(ALU_res),  // input wire [31 : 0] I0
  .I1(Data_in),  // input wire [31 : 0] I1
  .I2(c0),  // input wire [31 : 0] I2
  .I3(Imm_out),  // input wire [31 : 0] I3
  .o(Wt_data),         // output wire [31 : 0] o
  .s(MemtoReg)    // input wire [1 : 0] s
);
add_32_0 add0 (
  .a(32'h00000004),  // input wire [31 : 0] a
  .b(Q),  // input wire [31 : 0] b
  .c(c0)  // output wire [31 : 0] c
);
add_32_0 add1(
.a(Q),
.b(Imm_out),
.c(c1));
MUX2T1_32_0 Mu2t1 (
  .s((~ALU_zero&BranchN)|(ALU_zero&Branch)),    // input wire s
  .I0(c0),  // input wire [31 : 0] I0
  .I1(c1),  // input wire [31 : 0] I1
  .o(o1)    // output wire [31 : 0] o
);
MUX4T1_32_0 Mux4t1 (
  .I0(o1),  // input wire [31 : 0] I0
  .I1(c1),  // input wire [31 : 0] I1
  .I2(ALU_res),  // input wire [31 : 0] I2
  .I3(o1),  // input wire [31 : 0] I3
  .o(pc_next),         // output wire [31 : 0] o
  .s(Jump)    // input wire [1 : 0] s
);
RV_Int Rv_Int1(
.clk(clk),
.reset(rst),
.INT(INT0),
.ecall(ecall),
.mret(mret),
.ill_instr(ill_instr),
.pc_next(pc_next),
.pc(pc)
    );
    
MUX2T1_32_0 Mu2t0 (
  .s(ALUSrc_B),    // input wire s
  .I0(Rs2_data),  // input wire [31 : 0] I0
  .I1(Imm_out),  // input wire [31 : 0] I1
  .o(B)    // output wire [31 : 0] o
);
 ALU_more_0 ALU  (
.A(Rs1_data),
.B(B),
.ALU_operation(ALU_Control),
.res(ALU_res),
.zero(ALU_zero)
    );
RegFile_0 Regs (
  .clk(clk),            // input wire clk
  .rst(rst),            // input wire rst
  .RegWrite(RegWrite),  // input wire RegWrite
  .Rs1_addr(inst_field[19:15]),  // input wire [4 : 0] Rs1_addr
  .Rs2_addr(inst_field[24:20]),  // input wire [4 : 0] Rs2_addr
  .Wt_addr(inst_field[11:7]),    // input wire [4 : 0] Wt_addr
  .Wt_data(Wt_data),    // input wire [31 : 0] Wt_data
  .Rs1_data(Rs1_data),  // output wire [31 : 0] Rs1_data
  .Rs2_data(Rs2_data),  // output wire [31 : 0] Rs2_data
  .x0(x0),
  .ra(ra),
  .sp(sp),
  .gp(gp),
  .tp(tp),
  .t0(t0),
  .t1(t1),
  .t2(t2),
  .s0(s0),
  .s1(s1),
  .a0(a0),
  .a1(a1),
  .a2(a2),
  .a3(a3),
  .a4(a4),
  .a5(a5),
  .a6(a6),
  .a7(a7),
  .s2(s2),
  .s3(s3),
  .s4(s4),
  .s5(s5),
  .s6(s6),
  .s7(s7), 
  .s8(s8),
  .s9(s9),
  .s10(s10),
  .s11(s11),
  .t3(t3),
  .t4(t4),
  .t5(t5),
  .t6(t6)
  );


REG32 PC (
  .clk(clk),  // input wire clk
  .rst(rst),  // input wire rst
  .CE(2'b1),    // input wire CE
  .D(pc),      // input wire [31 : 0] D
  .Q(Q)      // output wire [31 : 0] Q
);

assign PC_out = Q;
assign Data_out = Rs2_data;
assign ALU_out = ALU_res;
endmodule
```

#### 3.4集成替换

![image-20230529110050620](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529110050620.png)

#### 3.5仿真测试指令

##### 3.5.1 ALU仿真测试

![image-20230529110251327](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529110251327.png)

**依次验证ALU八种运算**

##### 3.5.2 PC寄存器测试

![image-20230529110348365](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529110348365.png)

**当 Branch=0，Jump=0 时，PC 每个周期会自增加 4；**

**当 Branch=1，Jump=0 的时候，条件跳转，ImmSel=2，取出的立即数会-4；**

**Branch=0，Jump=1 的时候，ImmSel=11，取出的立即数也-4**

### 四.思考题

#### 4.1扩展下列指令，数据通路将作如何修改：

​	**R-Type：	sra, sll,sltu**
​	**I-Type：	 srai, slli, sltiu**
​	**B-Type：	bne；**
​	**U-Type：    lui**

**sra：算术右移，将寄存器中的数值向右移动指定位数，在左边用符号位进行填充。**
**sll：逻辑左移，将寄存器中的数值向左移动指定位数，在右边用 0 进行填充。**
**sltu：无符号比较大小，若目标寄存器 rs1 中的无符号整数小于目标寄存器 rs2 中的无符号整数，则 rd 被设置为 1，否则设置为 0。**
**srai：算术右移，将寄存器中的数值向右移动指定位数，在左边用符号位进行填充。**
**slli：逻辑左移，将寄存器中的数值向左移动指定位数，在右边用 0 进行填充。**
**sltiu：无符号比较大小，若目标寄存器 rs1 中的无符号整数小于立即数则 rd 被设置为 1，否则设置为 0。**
**bne：如果目标寄存器 rs1 和 rs2 中的值不相等，则将 PC 修改为当前 PC 加上符号扩展后的立即数，否则顺序执行下一条指令。**
**lui：将立即数左移 12 位后存储到目标寄存器 rd 中的高 20 位，低 12 位则被设置为 0。**

**需要拓展ALU的功能单元，并增加ALU的控制单元的位数**

#### 4.2增加I-Type算术运算指令是否需要修改本章设计的数据通路？

**I-Type算数运算指令不需要修改数据通路**



## 实验4-2：CPU-设计控制器

### 一.实验目的

1.	**运用寄存器传输控制技术**
2.	**掌握CPU的核心：指令执行过程与控制流关系**
3. **设计控制器**
4. **学习测试方案的设计**
5. **学习测试程序的设计**

### 二.实验任务

#### 任务一·：用硬件描述语言设计实现控制器

- **根据Exp04-1数据通路及指令编码完成控制信号真值表**
- **此实验在Exp04-1的基础上完成，替换Exp04-1的控制器核**

#### 任务二·：设计控制器测试方案并完成测试

- **OP译码测试：R-格式、访存指令、分支指令，转移指令**

- **运算控制测试：Function译码测试**

  ![image-20230529111526393](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529111526393.png)

  

### 三.实验流程

#### 1.设计主控制模块

![image-20230529112207133](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529112207133.png)

```verilog
always @*begin
    case(OPcode)
    5'b01100: // R Alusrc- 0 reg 1 imm  Memto reg  0   00 ALU 01 mem 10 PC+4 11 IMM
        case(Fun)
         4'b0000: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b000;ill_instr= 1'b0;end//add 0010
         4'b0001: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0110};ImmSel=3'b000;ill_instr= 1'b0;end//sub 0110
         4'b0010: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1110};ImmSel=3'b000;ill_instr= 1'b0;end//sll 1110
         4'b0100: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0111};ImmSel=3'b000;ill_instr= 1'b0;end//slt 0111
         4'b0110: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1001};ImmSel=3'b000;ill_instr= 1'b0;end//sltu 1001
         4'b1000: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1100};ImmSel=3'b000;ill_instr= 1'b0;end//xor 1100
         4'b1010: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1101};ImmSel=3'b000;ill_instr= 1'b0;end//srl 1101
         4'b1011: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1111};ImmSel=3'b000;ill_instr= 1'b0;end//sra 1111
         4'b1100: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0001};ImmSel=3'b000;ill_instr= 1'b0;end//or 0001
         4'b1110: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000};ImmSel=3'b000;ill_instr= 1'b0;end//and 0000
         default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end//and 0000
        endcase
      5'b00100:// I Alusrc- 0 reg 1 imm  Memto reg  0   00 ALU 01 mem 10 PC+4 11 IMM
      //`define CPU_ctrl_signals {ALUSrc_B,MemtoReg,RegWrite,MemRW,Branch,BranchN,Jump,ALU_Control}
      case(Fun3)
      3'b000: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b001;ill_instr= 1'b0;end//addi  0010
      3'b010: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0111};ImmSel=3'b001;ill_instr= 1'b0;end//slti  0111
      3'b011: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1001};ImmSel=3'b001;ill_instr= 1'b0;end//sltiu 1001
      3'b100: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1100};ImmSel=3'b001;ill_instr= 1'b0;end//xori  1100
      3'b110: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0001};ImmSel=3'b001;ill_instr= 1'b0;end//ori   0001
      3'b111: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000};ImmSel=3'b001;ill_instr= 1'b0;end//andi  0000
      3'b001: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1110};ImmSel=3'b001;ill_instr= 1'b0;end//slli  1110
      3'b101: 
      case(Fun7)
      1'b0:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1101};ImmSel=3'b001;ill_instr= 1'b0;end//srli 1101
      1'b1:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1111};ImmSel=3'b001;ill_instr= 1'b0;end//srai 1111
      default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end
      endcase
      endcase     
      //`define CPU_ctrl_signals {ALUSrc_B,MemtoReg,RegWrite,MemRW,Branch,BranchN,Jump,ALU_Control}
      //Alusrc- 0 reg 1 imm  Memto reg  0   00 ALU 01 mem 10 PC+4 11 IMM
    5'b00000:begin `CPU_ctrl_signals = {1'b1,2'b01,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b001;ill_instr= 1'b0;end//lw
    5'b11001:begin `CPU_ctrl_signals = {1'b1,2'b10,1'b1,1'b0,1'b0,1'b0,2'b10,4'b0010};ImmSel=3'b001;ill_instr= 1'b0;end//jalr
    5'b01000:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b0,1'b1,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b010;ill_instr= 1'b0;end//sw
    5'b11000:
    case(Fun3)
    3'b000: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b0,1'b0,1'b1,1'b0,2'b00,4'b0110};ImmSel=3'b011;ill_instr= 1'b0;end//Beq
    3'b001: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b0,1'b0,1'b0,1'b1,2'b00,4'b0110};ImmSel=3'b011;ill_instr= 1'b0;end//Bne
    default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b1;end
    endcase
    5'b01101:begin `CPU_ctrl_signals = {1'b0,2'b11,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000};ImmSel=3'b000;ill_instr= 1'b0;end//lui
    5'b11011:begin `CPU_ctrl_signals = {1'b1,2'b10,1'b1,1'b0,1'b0,1'b0,2'b01,4'b0000};ImmSel=3'b100;ill_instr= 1'b0;end//jal
    5'b01000:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b0,1'b1,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b010;ill_instr= 1'b0;end//sw
     5'b11100:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end
    default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end//and 0000
    endcase
    end
endmodule
```

#### 2.设计ALU操作译码

```verilog
    case(OPcode)
    5'b01100: // R Alusrc- 0 reg 1 imm  Memto reg  0   00 ALU 01 mem 10 PC+4 11 IMM
        case(Fun)
         4'b0000: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b000;ill_instr= 1'b0;end//add 0010
         4'b0001: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0110};ImmSel=3'b000;ill_instr= 1'b0;end//sub 0110
         4'b0010: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1110};ImmSel=3'b000;ill_instr= 1'b0;end//sll 1110
         4'b0100: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0111};ImmSel=3'b000;ill_instr= 1'b0;end//slt 0111
         4'b0110: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1001};ImmSel=3'b000;ill_instr= 1'b0;end//sltu 1001
         4'b1000: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1100};ImmSel=3'b000;ill_instr= 1'b0;end//xor 1100
         4'b1010: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1101};ImmSel=3'b000;ill_instr= 1'b0;end//srl 1101
         4'b1011: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1111};ImmSel=3'b000;ill_instr= 1'b0;end//sra 1111
         4'b1100: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0001};ImmSel=3'b000;ill_instr= 1'b0;end//or 0001
         4'b1110: begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000};ImmSel=3'b000;ill_instr= 1'b0;end//and 0000
         default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end//and 0000
        endcase
      5'b00100:// I Alusrc- 0 reg 1 imm  Memto reg  0   00 ALU 01 mem 10 PC+4 11 IMM
      //`define CPU_ctrl_signals {ALUSrc_B,MemtoReg,RegWrite,MemRW,Branch,BranchN,Jump,ALU_Control}
      case(Fun3)
      3'b000: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0010};ImmSel=3'b001;ill_instr= 1'b0;end//addi  0010
      3'b010: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0111};ImmSel=3'b001;ill_instr= 1'b0;end//slti  0111
      3'b011: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1001};ImmSel=3'b001;ill_instr= 1'b0;end//sltiu 1001
      3'b100: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1100};ImmSel=3'b001;ill_instr= 1'b0;end//xori  1100
      3'b110: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0001};ImmSel=3'b001;ill_instr= 1'b0;end//ori   0001
      3'b111: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000};ImmSel=3'b001;ill_instr= 1'b0;end//andi  0000
      3'b001: begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1110};ImmSel=3'b001;ill_instr= 1'b0;end//slli  1110
      3'b101: 
      case(Fun7)
      1'b0:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1101};ImmSel=3'b001;ill_instr= 1'b0;end//srli 1101
      1'b1:begin `CPU_ctrl_signals = {1'b1,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b1111};ImmSel=3'b001;ill_instr= 1'b0;end//srai 1111
      default:begin `CPU_ctrl_signals = {1'b0,2'b00,1'b1,1'b0,1'b0,1'b0,2'b00,4'b0000}; ImmSel=3'b000;ill_instr= 1'b0;end
      endcase
```

#### 3.仿真测试

##### 3.1 R型指令

![image-20230529112636320](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529112636320.png)

​																			**Fun3=3‘b000, Fun7=1’b0; 为 add 指令**

![image-20230529112728271](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529112728271.png)

​																			**Fun3=3‘b110, Fun7=1’b0; 为 or指令**

![image-20230529112952524](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529112952524.png)

​																							**Fun3=3‘b101, Fun7=1’b0; 为 srl指令**



##### 3.2 I型指令

![image-20230529113228148](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529113228148.png)

​																							**Fun3 = 3’b000,为 addi 指令**

![image-20230529113359239](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529113359239.png)

​																					**Fun3 =3b’001 时, 执行 slli 移位指令**

![image-20230529113449049](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529113449049.png)

​																					 **Opcode = 5’b00000,load 指令**

##### 3.3 UJ类型指令

![image-20230529113643340](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529113643340.png)

​																				**Opcode = 5’b11011 ， jal 指令**

#### 四.思考题

##### 4.1 单周期控制器时序体现在那里？

- **指令存取：单周期控制器通过指令存储器从内存中读取指令。具体地，单周期控制器使用 PC 寄存器存储下一条指令的地址，并将该地址送入指令存储器，指令存储器将对应地址中的指令读取出来并交给指令解码器进行解码。**
- **指令解码：单周期控制器从指令解码器中获取指令的操作码，并根据指令操作码生成相应的控制信号。控制信号包括了 ALU 的操作控制、寄存器堆读写控制、内存读写控制、分支跳转控制等。这些控制信号决定了整个单周期控制器执行的操作序列。**

- **执行阶段：在执行阶段，单周期控制器根据指令的操作码和控制信号进行运算或者数据传输。例如，对于加法指令，单周期控制器会在 ALU 中执行加法运算，并将结果写回到目标寄存器中。**

- **访存阶段：在访存阶段，单周期控制器会对内存进行读写操作。例如，在进行 load 指令时，单周期控制器需要从内存中读取数据，并将其写入目标寄存器。写入操作同理，需要将数据从源寄存器中读出，并写入内存中的指定地址。**

- **分支跳转：对于分支和跳转指令，单周期控制器需要根据控制信号修改 PC 寄存器中的值。例如，在进行 beq 指令时，单周期控制器需要比较两个寄存器中的值，如果相等则执行分支跳转。具体地，单周期控制器会计算跳转的地址并将其写入到 PC 寄存器中，然后指令存储器会从新的地址处继续获取指令并执行。**



##### 4.2 设计bne指令需要增加控制信号吗？

**不需要**

##### 4.3 扩展下列指令，控制器将作如何修改：

 **R-Type： sra, sll,sltu；**

 **I-Type： srai,slli,sltiu**

 **B-Type： bne；**

 **U-Type： lui；**

**此时用二级译码有优势吗？**

**修改控制器编码**

**二级译码相对于单级译码的优势主要在以下几个方面：**

- **灵活性更高：由于二级译码可以将指令的解码过程分为两个阶段，因此可以更加灵活地处理不同长度和复杂度的指令。例如，在处理 RISC-V 指令时，二级译码可以通过逻辑电路实现更高效的指令解码过程。**
- **缩短 CPU 时钟周期：在单级译码中，CPU 需要在一个时钟周期内完成指令的译码、取数据等所有操作，因此时钟周期常常需要较长，限制了 CPU 的运行速度。而二级译码可以将指令的译码过程分为两个时钟周期，从而缩短了 CPU 的时钟周期，提高了 CPU 的运行速度。**
- **缓解指令冲突：在单级译码中，如果多个指令需要同时访问某些共享资源（如寄存器、内存等），则会产生指令冲突，从而导致 CPU 运行效率降低。而二级译码可以在第一级译码时将操作数所在的寄存器编号提前读取出来，避免了指令冲突的发生，从而提高了 CPU 运行效率。**





## 实验4-3：CPU设计-指令集拓展

### 一.实验目的

1.	**运用寄存器传输控制技术**
2.	**掌握CPU的核心：指令执行过程与控制流关系**
3. **设计数据通路和控制器**
4. **设计测试程序**

### 二.实验任务

#### 任务一：重新设计数据通路和控制器，在lab4-2的基础上完成

**兼容lab4-1、 lab4-2的数据通路和控制器**
**替换lab4-1、 lab4-2的数据通路控制器核**
**扩展不少于下列指令**
	**R-Type：add, sub, and, or, xor, slt, sltu,srl, sra, sll；**
	**I-Type： addi, andi, ori, xori, slti, sltiu,srli,srai,slli,lw,jalr;**
	**S-Type：sw；**
	**B-Type：beq,bne；**
	**J-Type：  Jal；**
	**U-Type：lui；**

#### 任务二：设计指令集测试方案并完成测试	

### 三.实验流程

#### 3.1修改数据通路和控制顶层模块

```verilog
module 	  SCPU_ctrl_more( input[6:0]OPcode,	//OPcode
		        input[2:0]Fun3,	//Function
                                     input        Fun7,	//Function
		        input MIO_ready,	//CPU Wait
		        output reg [2:0]ImmSel,
		        output reg ALUSrc_B,
		        output reg [1:0]MemtoReg,
		        output reg [1:0]Jump,
		        output reg Branch,
		        output reg BranchN,
		        output reg RegWrite,
		        output reg MemRW,
		        output reg [3:0]ALU_Control,
		        output reg CPU_MIO
		        ); 			
endmodule

module 	Data_path_more( input clk,		//寄存器时钟
		  	input rst,		//寄存器复位
		 	input[31:0]inst_field,	//指令数据域[31:7]
		  	input ALUSrc_B,	//ALU端口B输入选择
		 	input [1:0]MemtoReg,	//Regs写入数据源控制
		  	input [1:0]Jump,		//J指令
		  	input Branch,		//Beq指令
		  	input BranchN,		//Bne指令
		  	input RegWrite,		//寄存器写信号
		 	input[31:0]Data_in,	//存储器输入
		 	input[3:0]ALU_Control,	//ALU操作控制
		 	input[2:0]ImmSel,	//ImmGen操作控制
						
		 	output[31:0]ALU_out,	//ALU运算输出
		 	output[31:0]Data_out,	//CPU数据输出
		 	output[31:0]PC_out	//PC指针输出
			);			
endmodule


```

#### 3.2重新编码指令，得到新的真值表

![image-20230529124311170](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529124311170.png)

#### 3.3 拓展ALU和ImmGen

![image-20230529124954725](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529124954725.png)

![image-20230529125021823](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529125021823.png)

```verilog
module ALU_more(
input [31:0] A,
input [3:0] ALU_operation,
input [31:0] B,
output reg[31:0] res,
output zero
);
wire [32:0] tempA;
wire [32:0] tempB;
assign tempA = {1'b0,A};
assign tempB = {1'b0,B};

always @ (A or B or ALU_operation) 
    case (ALU_operation) 
        4'b0000: res=A&B; //and
        4'b0001: res=A|B; //or
        4'b0010: res=A+B; //add
        4'b1100: res=A^B;  //xor
        4'b1101: res=A>>B[4:0];   //srl
        4'b1110: res=A<<B[4:0];   //sll
        4'b1111: res=A>>>B[4:0];   //sra
        4'b0110: res=A-B;   //sub
//        4'b100: res=~(A | B);
        4'b0111: res=(A < B) ?  32'h00000001 : 32'h00000000;  //slt
        4'b1001: res=(tempA < tempB) ?  32'h00000001 : 32'h00000000;  //sltu
        default: res=32'hx;
endcase
assign zero = (res==0)? 1: 0;
endmodule
```

```verilog
module ImmGen(
(* dont_touch = "true" *)input wire [2:0]ImmSel,
(* dont_touch = "true" *)input wire [31:0] inst_field,
(* dont_touch = "true" *)output reg [31:0] Imm_out
    );
    
    always @ * begin
    case(ImmSel)
    3'b001: Imm_out = {{20{inst_field[31]}},inst_field[31:20]};  // addi / lwi
    3'b010: Imm_out = {{20{inst_field[31]}},inst_field[31:25],inst_field[11:7]};  //s
    3'b011: Imm_out = {{19{inst_field[31]}},inst_field[31],inst_field[7],inst_field[30:25],inst_field[11:8],1'b0};  // beq
    3'b100: Imm_out = {{11{inst_field[31]}},inst_field[31],inst_field[19:12],inst_field[20],inst_field[30:21],1'b0};  //jal
    3'b000: Imm_out = {inst_field[31:12],12'h000};
    endcase  
    end
endmodule
```

#### 3.4 顶层设计

![image-20230529125116180](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529125116180.png)

```verilog
module SCPU_int(
input wire clk,
input wire rst,
input wire [31:0] inst_in,
input wire [31:0] Data_in,
input wire MIO_ready,
input wire INT0,
output wire CPU_MIO,
output wire MemRW,
output wire [31:0] Data_out,
output wire [31:0] Addr_out,
output wire [31:0] PC_out,
output wire [31:0] x0,
output wire [31:0] ra,
output wire [31:0] sp,
output wire [31:0] gp,
output wire [31:0] tp,
output wire [31:0] t0,
output wire [31:0] t1,
output wire [31:0] t2,
output wire [31:0] s0,
output wire [31:0] s1,
output wire [31:0] a0,
output wire [31:0] a1,
output wire [31:0] a2,
output wire [31:0] a3,
output wire [31:0] a4,
output wire [31:0] a5,
output wire [31:0] a6,
output wire [31:0] a7,
output wire [31:0] s2,
output wire [31:0] s3,
output wire [31:0] s4,
output wire [31:0] s5,
output wire [31:0] s6,
output wire [31:0] s7, 
output wire [31:0] s8,
output wire [31:0] s9,
output wire [31:0] s10,
output wire [31:0] s11,
output wire [31:0] t3,
output wire [31:0] t4,
output wire [31:0] t5,
output wire [31:0] t6
    );
  
wire [2:0] ImmSel;
wire ALUSrc_B;
wire [1:0] MemtoReg;
wire [1:0]Jump;
wire Branch;
wire RegWrite;
wire [3:0] ALU_Control;
wire BranchN;
wire ecall;
wire mret;
wire ill_instr;



SCPU_ctrl_int U1(
 .OPcode(inst_in[6:2]), 
 .Fun3(inst_in[14:12]),
  .Fun7(inst_in[30]),
  .Fun_ecall(inst_in[22:20]),
  .Fun_mret(inst_in[29:28]),
  .MIO_ready(MIO_ready), 
  .ImmSel(ImmSel),
  .ALUSrc_B(ALUSrc_B), 
  .MemtoReg(MemtoReg), 
  .Jump(Jump), 
  .Branch(Branch),
  .BranchN(BranchN),
  .RegWrite(RegWrite),
  .MemRW(MemRW), 
  .ALU_Control(ALU_Control), 
  .mret(mret),
  .ill_instr(ill_instr),
  .ecall(ecall),
  .CPU_MIO(CPU_MIO));
  
Datapath_0 U2(
  .clk(clk), 
  .rst(rst), 
  .inst_field(inst_in), 
  .Data_in(Data_in), 
  .ALU_Control(ALU_Control), 
  .ImmSel(ImmSel), 
    .MemtoReg(MemtoReg), 
    .ALUSrc_B(ALUSrc_B), 
    .Jump(Jump), 
    .Branch(Branch), 
    .BranchN(BranchN),
    .RegWrite(RegWrite),
    .mret(mret),
    .ill_instr(ill_instr),
    .INT0(INT0),
    .ecall(ecall), 
    .PC_out(PC_out), 
    .Data_out(Data_out), 
    .ALU_out(Addr_out),
      .x0(x0),
    .ra(ra),
    .sp(sp),
    .gp(gp),
    .tp(tp),
    .t0(t0),
    .t1(t1),
    .t2(t2),
    .s0(s0),
    .s1(s1),
    .a0(a0),
    .a1(a1),
    .a2(a2),
    .a3(a3),
    .a4(a4),
    .a5(a5),
    .a6(a6),
    .a7(a7),
    .s2(s2),
    .s3(s3),
    .s4(s4),
    .s5(s5),
    .s6(s6),
    .s7(s7), 
    .s8(s8),
    .s9(s9),
    .s10(s10),
    .s11(s11),
    .t3(t3),
    .t4(t4),
    .t5(t5),
    .t6(t6));
endmodule
```

#### 3.5 下板测试

![image-20230529132342250](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529132342250.png)

**0x1c: beq x7 x3 16 ,跳转指令，跳转到0x2c**

![拓展指令测试1](D:\FFOutput\拓展指令测试1.gif)

**0x34:bne x6 x5 8 ,跳转到0x3c**

![拓展指令测试2](D:\FFOutput\拓展指令测试2.gif)

### 四.思考题

#### 4.1指令扩展时控制器用二级译码设计存在什么问题？

1. **时间延迟增加：由于采用了二级译码的设计，所有的指令都需要在第二级译码阶段才能被正式解析，这可能导致 CPU 运行时需要增加一个时钟周期的时间延迟。**
2. **硬件复杂度增加：二级译码需要增加更多的逻辑门电路来完成指令的译码过程，因此会增加 CPU 的硬件复杂度，从而增加了制造成本和故障率等风险。**
3. **稳定性降低：二级译码可能会对 CPU 的稳定性产生影响，因为译码需要在两个时钟周期内完成，如果时序控制不好，容易产生指令串扰等问题，从而导致CPU出现错误。**

#### 4.2设计bne指令需要增加控制信号吗？

**增加一位控制信号**

#### 4.3设计srai时需要增加新的数据通道吗？

**不需要**



## 实验4-4：CPU设计-中断

### 一.实验目的

1.	**深入理解CPU结构**
3. **学习如何提高CPU使用效率**
3. **学习CPU中断工作原理**
4. **设计中断测试程序**

### 二.实验任务

#### 任务一：扩展实验CPU中断功能

**修改设计数据通路和控制器**：
**1.修改或替换Exp04-3的数据通路及控制器**
**2.兼容Exp04-3数据通路增加中断通路**
**3.增加中断控制**

**扩展CPU中断功能**：
**1.非法指令中断；**
**2.ecall**

#### 任务二：设计CPU中断测试方案并完成测试

### 三.实验流程

#### 3.1DataPath修改

- **CPU复位时，MEPC=PC=0x00000000**
- **修改PC模块增加**
- **mtvec寄存器型变量，中断和异常触发PC转向中断地址**
- **相当于硬件触发Jal，用mret返回**
- **mepc寄存器，中断和异常返回PC的地址**
- **增加控制信号INT、mret、ecall、ill_instr**

```verilog
module RV_Int(
    //input
    input wire clk,
    input wire reset,
    input wire INT,
    input wire ecall,
    input wire mret,
    input wire ill_instr,
    input wire[31:0] pc_next,
    //output
    output reg[31:0] pc
);

    reg[31:0] mepc;
    reg interr;  
    reg preINT;
initial begin
interr<=1;
end
    always @(posedge clk) begin
      if(reset)begin
        mepc <= 32'h0000_0000;
        pc   <= 32'h0000_0000;
        interr <= 1; 
      end
      else if(INT && interr)begin
       if(preINT == 0)begin
        pc   <= 32'h0000_000c;
        mepc <= pc_next;
        interr <= 0;   
        end
      end
      else if(ecall && interr)begin
        pc   <= 32'h0000_0008;
        mepc <= pc_next;
        interr <= 0;
      end
      else if(ill_instr && interr)begin
        pc   <= 32'h0000_0004;
        mepc <= pc_next;
        interr <= 0;
      end
      else if(mret && !interr)begin 
        pc <= mepc;
        interr <= 1;
      end
      else begin
          pc <= pc_next;
          interr <= interr;
      end
    preINT = INT;
    end
endmodule

```

#### 3.2控制器修改

- **简洁模式**
- **增加mret、ecall指令以及非法指令的处理**
- **中断请求信号触发PC转向，在Datapath模块中修改**

```verilog
always @*begin
    case (OPcode)
    5'b11100:
    case(Fun_mret)
    2'b11: begin mret= 1'b1;end
    default:begin mret = 1'b0;end
    endcase
    default:begin mret = 1'b0;end
    endcase
    end
```

#### 3.3 下板中断测试

![image-20230529132208904](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529132208904.png)

![image-20230529155154069](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529155154069.png)

![image-20230529132147941](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529132147941.png)



1. **ecall**

   ![image-20230529132124574](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529132124574.png)

   0x74遇到ecall指令，因此发生中断，跳转到0x8,然后执行 jal x0 216 ,跳转到0xe0,然后0xe8遇到mret指令,回到0x78下一条指令继续执行

   ![ecall](D:\FFOutput\ecall.gif)

2. **非法指令**

   ![image-20230529132110459](C:\Users\86184\AppData\Roaming\Typora\typora-user-images\image-20230529132110459.png)

   在0x2c处遇到非法指令，发生中断，跳转到0x4，然后执行jal x0 196 ,跳转到0xc8 ,然后到0xd0执行mret回到0x2c下一条指令

   ![违法指令](D:\FFOutput\违法指令.gif)
