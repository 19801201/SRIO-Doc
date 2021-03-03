# SRIO-Doc

## RapidIO层次构建
SRIO模块由三层构建而成。
### 逻辑层 确定终端处理传输的协议，包括包的格式。
### 传输层 定义了在系统中正确路由信息包的寻址方案。
### 物理层 包含设备级的接口信息，如电气特性、错误管理数据和基本的流量控制数据。
### 传输层与逻辑层和物理层是上下兼容的。

逻辑层包含终端处理传输（transaction）的必要信息，如传输类型、大小、物理地址。
传输层包含系统中终端相互传输包（packet）的信息，如寻址。
物理层包含物理设备之间相互传递包（packet）时所需的信息，如电接口，流的控制。

## 物理层 1x/4x LP-Serial（长浮点串口） 规格

现在有两种SRIO贸易协会认可的物理层规格:
1、 8/16 LP-LVDS
2、 1x/4x LP-Serial
第一种规格是点对点同步时钟源DDR接口；第二种规格是点对点，交流耦合，时钟恢复接口。而且两种规格不兼容。
SRIO遵从第二种规格，即1x/4x LP-Serial规格，SRIO中的串行/解串技术也是由这种规格分配的。
该规格适用于4个频率点，即1.25,2.5,3.125和5Gbps，这定义了每个I/O差分对的总带宽。
有一个8位/ 10位编码方案，确保时钟恢复电路的充足的数据转换。由于8位/ 10位编码方案的开销，每个差分对各自的有效数据带宽是1，2，2.5，4 Gbps。SRIO只同时为1X和4X的port指定频率。
一个1X port被定义为一个TX（transanction）和一个RX（receive）的差分对。类似于一个IO口的差分对。4 X port就是4个1X的组合。一个4X port也可以被配置为4个1X port。
SRIO提供了支持从1G到16G带宽的可升级接口。
下图显示了怎样连接两个1X设备（或两个4X设备），一个设备的positive transmit data line (TDx) [高电平有效传输数据线TDx]和另一个设备的positive receive data line (RDx)相连，低电平有效传输数据线 TDx和低电平有效接收数据线RDx相连。
![Image](https://github.com/19801201/Image/blob/main/img1.png)

## SRIO 传输规范
SRIO  中数据传输的任意两端都可以充当M(主器件)或S(从器件)。
数据发起的一方称为M,数据接收的一方就成为S。

下图为通过ireq  接口发送 NWRITE 包的情况。
![Image](https://github.com/19801201/Image/blob/main/img2.png)
图中包头为  0x3854205198877600 
第52-55 位 为0101 ,  第 51-48 位为0100  。
52-55 位 位为 FTYPE, 则 FTYPE 为 5。
第 51-48 位为 TTYPE，则 TTYPE 为 4。

下图为  FPGA 通过 SWRITE(流写) 向 DSP 端的内存发送数据，一次数据写完后将会发送  DOORBELL 中断提示  DSP  处理数据。
![Image](https://github.com/19801201/Image/blob/main/img3.png)
![Image](https://github.com/19801201/Image/blob/main/img4.png)

### 在传输时需注意 :
首先会发送一个  64 位的 包头 (head_reg), 
然后发送一个 64 位的数据，数据对齐到 8 个字节。SWRITE  可以携带的包是 8-256 个字节 。

### 包头  head_reg 定义如下图

![Image](https://github.com/19801201/Image/blob/main/img5.png)

前 8 位为 Tid,                                                      tid_reg   
接着4位是包的类型(本例是WRITE)， 					 			   SWRITE
接着4位是默认值0,												   4’h0
再接着1位是默认值0。											   1’b0
然后的两位是 Prio_req (优先级寄存器 )。                              Prio_req
再接着以为是 crf (默认值是0) 。								       1’b0
再接着的8位是用来指定所携带的数据的大小的。但是在 SWRITE 中不能指定所携带的数据的大小。所以 SWRITE 包不能通过包的内容判断包的实际长度。接收端是通过包尾符来判断包的结束的。(这些都通过SRIO的物理层来完成的)                       8’h00

剩下的  36  位是用来指定对方的内存地址的。                          dsp_address_r

发送完64位的包头后，紧接着就是64位的数据  。  (最多可以发送  256 bit 的数据，即发送4个64位的数据) 。
发完一次数据，若想在发送新的数据，那么就需要分包组包操作。需要相应的硬件电路来配合。(FPGA  中) 
DSP  中的分包组包是自动完成的  。

