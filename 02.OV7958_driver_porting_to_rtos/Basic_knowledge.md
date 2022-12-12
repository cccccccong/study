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

* **两种 suspend mode：**

  - hardware standby:
    硬件待机，PWDN 拉高，此时内部设备时钟停止，所有内部计数器复位，寄存器保持，大多数数字电路保持在断点状态.
  - software standby:
    软件待机，通过 SCCB 接口发送命令执行软件待机，暂停内部电路活动，但不停止设备时钟，所有寄存器内容都保持在待机模式.

* **图像输出格式**

| 功能     | 格式 | 分辨率  | 场率/帧率   | 输入时钟     |
| -------- | ---- | ------- | ----------- | ------------ |
| 模拟输出 | NTSC | 648x488 | 60 fields/s | 24.545452MHz |
| DVP      | YUV  | 640x480 | 可调节的    | 24MHz        |

