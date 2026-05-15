---
title: 'DNESP32S3 BOX3 介绍'
sidebar_position: 3

---

# DNESP32S3 BOX3介绍

<img src={require('./img/dnesp32s3-box3-show.png').default} alt="dnesp32s3 box3 show" width="474" />

DNESP32S3 BOX3是正点原子以ESP32S3为核心推出的开发板，主要用于让客户快速搭建AI硬件原型机，同时非常适合用于学习ESP32S3芯片的开发。

## 外观

<img src={require('./img/dnesp32s3-box3-show-six-back.png').default} alt="dnesp32s3 box3 show six back" width="474" />

DNESP32S3 BOX3整体采用定制的外壳对主板以及屏幕进行固定和保护，背面是一块定制的透明亚克力，透过亚克力可以看到，DNESP32S3 BOX3 内部主板采用了一体化设计结构，从而实现电源供给与数据交互。

DNESP32S3 BOX3整理小巧、美观，其与其内部的底板的三维尺寸如下

| 开发板            | 长（mm） | 宽（mm） | 高（mm） |
| -------------- | ----- | ----- | ----- |
| DNESP32S3 BOX3 | 64.30 | 47.60 | 15.40 |

## DNESP32S3 BOX3 硬件基本参数

DNESP32S3 BOX3 硬件基本参数如表所示：

| 项目   | 说明                 |
| ---- | ------------------ |
| 产品型号 | ATK-DNESP32S3B3 V1 |
| 芯片   | ESP32S3-R8         |
| 工作电压 | 5V（USB）            |
| 工作电流 | 75mA~276mA（@5V）①   |
| 工作温度 | 0℃~+70℃            |

注①：75mA 对应 CPU 在复位情况下，裸板的工作电流； 276mA 对应 CPU 满负荷运行时的工作电流。

## DNESP32S3 BOX3 硬件资源分布

DNESP32S3 BOX3搭载了丰富硬件外设，方便用户开发和使用，其硬件资源分布如下所示(资源介绍不分先后)：
<img src={require('./img/dnesp32s3-box3-interface.png').default} alt="dnesp32s3 box3 show six back" width="1000" />

| 资源            | 数量  | 说明                                                                         |
| ------------- | --- | -------------------------------------------------------------------------- |
| 主控 ESP32S3-R8 | 1个  | Xtensa® 32 位 LX7 双核处理器，主频高达 240MHz<br />384 KB ROM<br />512 KB SRAM        |
| Flash芯片       | 1个  | 采用华邦的16MB芯片                                                                |
| 电源指示灯（PWR）    | 1个  | 支持蓝色，上电后常亮                                                                 |
| 0805红蓝双色LED灯  | 1个  | 支持红蓝双色                                                                     |
| 复位按键          | 1个  | 用于芯片 & LCD 的复位                                                             |
| 功能按键          | 3个  | K2（通过IO扩展芯片控制）<br />K1（通过IO扩展芯片控制）<br /> K0（通过主控芯片控制）                      |
| 音频编解码芯片       | 2个  | ES8311用于输出，ES7210用于采集。该方案用于AEC回声消除（与小智对话可随时通过语音打断）                         |
| 录音咪头（MIC）     | 1个  | 模拟麦克风，用于录音                                                                 |
| 功放芯片          | 1个  | NS4150B，将音频信号放大后驱动扬声器发声                                                    |
| 板载扬声器         | 1个  | 2520腔体喇叭，用于播放音乐或者视频声音                                                      |
| IO扩展芯片        | 1个  | AW9523BTQR，用于扩展IO引脚                                                        |
| 三轴传感器         | 1个  | SC7A20H，用于测量翻滚角、 俯仰角等数据                                                    |
| LCD 接口        | 1个  | 可接正点原子 2.4 寸带触摸 LCD 模块                                                     |
| Type-C 接口     | 1个  | 支持 USB OTG（默认为 Device ）<br />主要用于设备的电源供给以及进行 USB 通讯<br />VSCode 使用该接口与设备连接 |
| 排针引出接口        | 1个  | 共引出 22 个 GPIO，均已做等长处理                                                      |
| 摄像头接口         | 1个  | 分别连接至 DNESP32S3 BOX3 的 摄像头电路，支持如下摄像头：GC0308                                |
| PH2.0 接口      | 2个  | 可对外 5V 供电，并支持 UART、IIC、GPIO 等外设功能<br />                                    |
| TF 卡槽         | 1个  | 用于外接 TF 卡<br />连接至 DNESP32S3 BOX3 的 SD 接口                                  |
| 板载天线          | 1个  | CA-C03 <br />用于释放热点信号与连接WiFi                                               |
