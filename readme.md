
>+ 本课设主要研究了多相滤波器的原理、实现和应用。相关的原理主要参考：
《软件无线电技术与应用》 杨小牛 北京理工大学出版社 2010.4
>+ 课设时间：2016年3月
>+ 整理原因：回忆一下以前认真做过的试验项目，毕竟该项目涉及到多相滤波器的各种来龙去脉实现。

1. 原理解释.pptx 是课设答辩PPT，含注释，主要是原理解释和仿真、实物运行结果相关解释。

>>这里重点介绍一下自己对多相滤波的看法：多相滤波器并不会增加滤波性能，也不会减少FPGA逻辑单元的占用，它的优势是在FIR滤波过程中----由于按照多相结构计算，各相的累加延时减少了，因此 **在相同的FPGA时钟周期下可以实现的总体FIR阶数更长**！！也就是说在相同的滤波性能要求（FIR阶数固定）的情况下，对FPGA的时钟要求较小。另外，**信号分解-->分相并行处理-->重建原信号** 以及 **采样率变换** 是多相滤波结构的典型应用。

2. 根目录下的matlab代码分别演示了所涉及的原理的具体实现方式。
各个仿真文件的逻辑结果为：信号产生--滤波--信号输出 -- 时频对比绘制。
 + proto_filter.m 和 polyphase_filter.m 是原型滤波器及其多相结构实现。
 + filter_*_sample.m 是上下采样结构的正常实现示例（都是在高采样率下进行滤波，滤波器阶数高时在硬件实现中对FPGA时钟要求较高），其频谱变化结论参考PPT。
 + polyphase_filter_*_sample.m 是上下采样的多相实现结构 **验证**，结果图像和正常实现相同。
 + polyphase_*_real.m 是多相结构实现的 **实数信道化收发机**，是多相滤波器的应用，分别对应原书籍中的3.3和3.5节。
 + polyphase_*_complex.m 是是多相结构实现的 **复数信道化收发机**，是多相滤波器的应用，分别对应原书籍中的3.3和3.5节。相对于实数收发机，复数收发机的频谱利用率更高。
 + cosSignalGen 和 myPsdCal 就是简单的小函数，打开一看就知道是干啥的了。

> 实现过程中多相结构收发机参考书上的结构完成，但是发现存在一定的问题，主要发射的数据和接收的数据不对应，做了一些变换调整后一致了。具体的数学原理不详（书上页没提到这个问题），但是校正方法在m文件里面有介绍！！

3. SimAssitant 目录下主要是在编写FPGA代码时用到的一些辅助matlab脚本，主要是DDS信号产生的ROM数据生成、滤波器系数生成等，详细实现看.m代码、、、我也不记得了。
4. VERILOG 目录下是使用 Quartus II 13.0.1.232 编写的verilog仿真工程。其顶层实例out_da_data实现根据输入的key[2..0]实现八种状态的数据输出（65阶滤波器滤波信号及其原始信号、169阶多相滤波器滤波信号及其原始信号、169阶多相滤波器抽取/插值信号及其原始信号），主要包括四个实例分别为：
 + data_fir 信号及其65阶滤波输出；
 + polypase_fir 信号及其169阶多相滤波器滤波信号输出；
 + decimation 信号及其抽取信号输出；
 + interpolation 信号及其插值信号输出；

>其中涉及的wave_add是叠加的多个COS信号，包含多个频率点用于演示滤波效果，其调用sin_dds产生各个SIN信号。而radix_fir_65以及fir/目录下的各个滤波器的实现，其系数使用 SimAssitant 目录下的matlab脚本产生，转为verilog的代码方法请参考[开源FIR设计](http://www.cdta.dz/products/mcm/).

 + 在VERILOG\simulation\modelsim下包含的 *.vt 文件是仿真testbench文件，可以直接在Quartus 下使用modelsim时序仿真，主要包括对以上四个实例分别仿真以及对顶层实例仿真的tb，可以在设置中选择。从底层到顶层没完成一个模块都会有一个相应的仿真文件。

5. ISEprj 目录下是使用XILINX开发板进行实物运行所需的工程，在ISE14.0下建立。这么做是因为实验室只有这个开发板包含AD/DA模块。但是逻辑完全是VERILOG目录下的代码逻辑，只是需要IO映射而已。


本文地址：https://github.com/HeLiangHIT/polyphase_filter_prj




