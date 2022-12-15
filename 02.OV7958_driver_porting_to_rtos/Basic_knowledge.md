[toc]

# 移植 OV7958 驱动到 rtos —— 基础知识

## datasheet

### 名词、缩写等

- VGA : Video Graphics Array
- DV : Digital Video
- 摄像机制式 PAL 和 NTSC：NTSC 即正交平衡调幅制，PAL 为逐行倒像正交平衡调幅制。
- SCCB ：Serial Camera Control Bus, 串行摄像机控制总线协议。
- ov7958 slave addresses are 0x80 for write and 0x81 for read.

### sensor

- power supply, 内部调节器输出电压由寄存器 0x3612[2]控制.

- **power up sequence, 上电顺序：**
  1. 先给 DOVDD 供电到稳定再给 AVDD 供电.
  2. PWDN 拉高，clock 不必要.
  3. 在上电周期中， PWDN 必须为高.
  4. 要想拉低 PWDN，先稳定电源.(AVDD 到 PWDN 拉低保持至少 5ms)
  5. 在访问传感器的寄存器之前，至少保持 2ms MCLK.
  6. 主机可以在整个周期内访问 SCCB 总线，PWDN 拉低 20ms 后，主机可以访问传感器的寄存器以初始化传感器.

* **reset**
  传感器上电会把所有寄存器复位到默认值，把 0x0103[0]写 1 同样可以 reset，reset 需要至少保持 2ms.

- **两种 suspend mode：**

  - hardware standby:
    硬件待机，PWDN 拉高，此时内部设备时钟停止，所有内部计数器复位，寄存器保持，大多数数字电路保持在断点状态.
  - software standby:
    软件待机，通过 SCCB 接口发送命令执行软件待机，暂停内部电路活动，但不停止设备时钟，所有寄存器内容都保持在待机模式.

- **图像输出格式**

| 功能     | 格式 | 分辨率  | 场率/帧率   | 输入时钟     |
| -------- | ---- | ------- | ----------- | ------------ |
| 模拟输出 | NTSC | 648x488 | 60 fields/s | 24.545452MHz |
| DVP      | YUV  | 640x480 | 可调节的    | 24MHz        |

## 调试注意

1. 查看原理图的供电通路，务必确认供电正常，确认 I2C 总线上有上拉电阻。
2. 查看 datasheet 中的上电时序介绍，上电时一般需要管理三个 PIN，分别为 power,pwdn,rest。根据时序进行操作 power，pwdn，reset pin，不然可能无法正常上电启动。
3. 配置好 I2C 总线，I2C 初始化后在没有数据时 IO 应为高电平。
4. 写摄像头驱动读写寄存器函数，注意设备地址和传输数据的有效位，比如 ov7958 就是 16bit 寄存器地址，先传输高 8 位，后传输低 8 位。
5. 尝试读取摄像头的 ID,如果硬件和 I2C 配置没有问题，此时应该可以读取到 id 值。
6. camera 初始化完成后，测量 MCLK 和 PCLK，查看时钟是否正常。
7. 根据数据手册中的接口类型和输出格式来配置参数，dvp 接口的话包括 hsync,vsync,pclk 的极性等，注意 dvp 接口是 8bit 还是 10bit 以及与开发板是怎样相连的。
8. 采出的数据使用 7yuv 打开，看图像是否正常并分析原因。

## 扩展

### 接口类型

- DVP：Digital Video Port, 8/10 bit.
- MIPI：MobileIndustry Processor Interface.
- DVP 和 MIPI 总结：
  - 优点不同:
    1. DVP 接口：DVP 是并口传输，速度较慢，传输的带宽低。
    2. MIPI 接口：MIPI 是差分串口传输，速度快，抗干扰。
  - 特点不同:
    1. DVP 接口：使用需要 PCLK\sensor 输出时钟、MCLK（XCLK）\外部时钟输入、VSYNC\帧同步、HSYNC\行同步、D[0：11]\并口数据可以是 8/10/12bit 数据位数大小。
    2. MIPI 接口：主流手机模组现在都是用 MIPI 传输，传输时使用 4 对差分信号传输图像数据和一对差分时钟信号；最初是为了减少 LCD 屏和主控芯片之间连线的数量而设计的，后来发展到高速了，支持高分辨率的显示屏，现在基本上都是 MIPI 接口了。
  - 电源组成不同:
    1. DVP 接口：使用 1.5V 或更高，不同厂家的设计不同，1.5V 可能由 sensor 模组提供或外部供给，可以使用外部供电则建议使用外部供电。
    2. MIPI 接口：VDDIO（IO 电源），AVDD（模拟电源），DVDD（内核数字电源），不同 sensor 模组的摄像头供电不同，AVDD 有 2.8V 或 3.3V 的；

参考 [DVP 和 MIPI 区别](https://blog.csdn.net/weixin_44174312/article/details/126950158)

### 格式说明

#### RAW，YUV，RGB
TODO：
