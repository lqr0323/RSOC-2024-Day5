# RSOC-2024-Day5  
本文章是基于2024-RSOC暑期夏令营的第五天的  **软件包与组件**  的学习中的一些分享。
## 简介

AHT10 软件包提供了使用温度与湿度传感器 `aht10` 基本功能，并且提供了软件平均数滤波器可选功能。并且本软件包新的版本已经对接到了 Sensor 框架，通过 Sensor 框架，开发者可以快速的将此传感器驱动起来。若想查看**旧版软件包**的 README 请点击[这里](README_OLD.md)。

基本功能主要由传感器 `aht10` 决定：在输入电压为 `1.8v-3.3v` 范围内，测量温度与湿度的量程、精度如下表所示

| 功能 | 量程 | 精度 |
| ---- | ---- | ---- |
| 温度 | `-40℃ - 85℃` |`±0.5℃`|
| 湿度 | `0% - 100%` |`±3%`|

## 支持情况

| 包含设备 | 温度 | 湿度 |
| ---- | ---- | ---- |
| **通信接口** |          |        |
| IIC      | √        | √      |
| **工作模式**     |          |        |
| 轮询             | √        | √      |
| 中断             |          |        |
| FIFO             |          |        |

## 使用说明

### 依赖

- RT-Thread 4.0.0+
- Sensor 组件
- IIC 驱动：aht10 设备使用 IIC 进行数据通讯，需要系统 IIC 驱动支持

### 获取软件包

使用 aht10 软件包需要在 RT-Thread 的包管理中选中它，具体路径如下：

```
RT-Thread online packages  --->
  peripheral libraries and drivers  --->
    sensors drivers  --->
            aht10: digital humidity and temperature sensor aht10 driver library. 
                         [ ]   Enable average filter by software         
                               Version (latest)  --->
```

**Enable average filter by software**：选择后会开启采集温湿度软件平均数滤波器功能。

**Version**：软件包版本选择，默认选择最新版本。

### 使用软件包

aht10 软件包初始化函数如下所示：

```
int rt_hw_aht10_init(const char *name, struct rt_sensor_config *cfg)；
```

该函数需要由用户调用，函数主要完成的功能有，

- 设备配置和初始化（根据传入的配置信息配置接口设备）；
- 注册相应的传感器设备，完成 aht10 设备的注册；

#### 初始化示例

```c
#include "sensor_asair_aht10.h"
#define AHT10_I2C_BUS  "i2c4"

int rt_hw_aht10_port(void)
{
    struct rt_sensor_config cfg;

    cfg.intf.dev_name  = AHT10_I2C_BUS;
    cfg.intf.user_data = (void *)AHT10_I2C_ADDR;
    
    rt_hw_aht10_init("aht10", &cfg);

    return RT_EOK;
}
INIT_ENV_EXPORT(rt_hw_aht10_port);
```
## 打开RW007的WIFI连接  
首先，右键打开env，在命令窗口输入 ```menuconfig```,进入```Hardware Drivers Config --->Onboard Peripheral Drivers ---> Enable Rw007.....```  
修改如下配置：  
```
(90) RW007 CS pin index
(29) RW007 BOOT0 pin index (same as spi clk pin)
(90) RW007 BOOT1 pin index (same as spi cs pin)
(107) RW007 INT/BUSY pin index
(111) RW007 RESET pin index
```
修改完记得在env中更新软件包：  
```
kpgs --update
```
连接上开发板后，打开串口软件，按下 ```reset``` ,然后输入 ```wifi``` ,出现  
```
msh />wifi
wifi
wifi help
wifi scan [SSID]
wifi join [SSID] [PASSWORD]
wifi ap SSID [PASSWORD]
wifi disc
wifi ap_stop
wifi status
wifi smartconfig
```
输入 ```wifi scan```,可以查看扫描的wifi   
使用```wifi join```,可以加入wifi
## 阿里云MQTT  
名词释义：

Publisher - 发布者

Broker - 代理（服务端）

Subscriber - 订阅者

Topic - 发布/订阅的主题

流程概述：上图中，各类传感器的角色是发布者(Publisher)。譬如，湿度传感器和温度传感器分别向接入的MQTT Broker中（周期性）发布两个主题名为"Moisture"（湿度）和"Temp"（温度）的主题；当然，伴随着这两个主题共同发布的，还有湿度值和温度值，被称为“消息”。几个客户端的角色是订阅者Subscriber，如手机APP从Broker订阅了"Temp"主题，便能在手机上获取到温度传感器Publish在Broker中的温度值。

补充说明：

发布者和订阅者的角色并非是固定的，而是相对的。发布者也可以同时从Broker订阅主题，同理，订阅者也可以向Broker发布主题；即发布者可以是订阅者，订阅者也可以是发布者。
Broker可以是在线的云服务器，也可以是本地搭建的局域网客户端；按照需求，实际上Broker自身也会包含一些订阅/发布主题的功能。
更多参考资料，请前往MQTT中文网或MQTT官网查阅。  
MQTT（Message Queuing Telemetry Transport）是一种轻量级、基于发布-订阅模式的消息传输协议，适用于资源受限的设备和低带宽、高延迟或不稳定的网络环境。它在物联网应用中广受欢迎，能够实现传感器、执行器和其它设备之间的高效通信。

特点：

- **轻量级：**物联网设备通常在处理能力、内存和能耗方面受到限制。MQTT 开销低、报文小的特点使其非常适合这些设备，因为它消耗更少的资源，即使在有限的能力下也能实现高效的通信。

- **可靠：**物联网网络常常面临高延迟或连接不稳定的情况。MQTT 支持多种 QoS 等级、会话感知和持久连接，即使在困难的条件下也能保证消息的可靠传递，使其非常适合物联网应用。

  三种Qos等级：最多一次（QoS0）、至少一次（QoS1）、仅一次（QoS2）

- **便于拓展：**如果有设备需要获取某个传感器的消息，只需要订阅这个主题就好了。

### 1.2.2 运行框架

**Client：**客户端，即我们使用的设备。

使用MQTT的程序或设备。客户端总是通过网络连接到服务端。它可以

- 发布应用消息给其它相关的客户端。
- 订阅以请求接受相关的应用消息。
- 取消订阅以移除接受应用消息的请求。
- 从服务端断开连接。

**Server：**服务端

作为发送消息的客户端和请求订阅的客户端之间的中介。服务端

- 接受来自客户端的网络连接。
- 接受客户端发布的应用消息。
- 处理客户端的订阅和取消订阅请求。
- 转发应用消息给符合条件的已订阅客户端。

**Topic Name：**主题名

附加在应用消息上的一个标签，服务端已知且与订阅匹配。服务端发送应用消息的一个副本给每一个匹配的客户端订阅。

**Subscription：** 订阅

订阅相应的主题名来获取对应的信息。

**Publish：**发布

在对应主题上发布新的消息。  
![image-20240725132821170](https://github.com/lqr0323/RSOC-2024-Day5/blob/main/MQTT%E8%BF%90%E8%A1%8C%E6%A1%86%E6%9E%B6.png))

### 1.2.3 阿里云搭建

平台：[https://www.aliyun.com/product/iot/iot_instc_public_cn](https://www.aliyun.com/product/iot/iot_instc_public_cn)

进入后进行注册（如果你是新账户的话）并登录，然后选择管理控制台

![image-20240725184110285](Pictures/image-20240725184110285.png)

在管理控制台下选择公共实例

![image-20240725184217401](Pictures/image-20240725184217401.png)

点击创建产品

<img src="Pictures/image-20240725184400238.png" alt="image-20240725184400238" style="zoom:50%;" />

按要求输出，产品名称可以随意填写

![image-20240725184517799](Pictures/image-20240725184517799.png)

然后返回，点击设备，创建设备

![image-20240725184835993](Pictures/image-20240725184835993.png)

创建完成后可以新建一个物模型变量

![image-20240725185053198](Pictures/image-20240725185053198.png)

![image-20240725185216703](Pictures/image-20240725185216703.png)

然后点击发布上线，然后就可以在功能定义这里看到刚刚定义的功能了。记得在Topic类列表中修改自定义user/get的权限改为订阅和发布，这样子我们才能通过这个Topic进行测试。

![image-20240725185630684](Pictures/image-20240725185630684.png)

调试功能，在课上讲。然后就是配置menuconfig。

打开RW007

![MQTT配置1](Pictures/MQTT配置1.png)

在软件包中找到阿里云的软件包

![MQTT配置2](Pictures/MQTT配置2.png)

使能后需要修改里面的一些参数

![MQTT配置3](Pictures/MQTT配置3.png)

在阿里云中找到这些参数并修改进去

![image-20240725192141241](Pictures/image-20240725192141241.png)

![image-20240725192218149](Pictures/image-20240725192218149.png)

然后再打开ENV末尾的使能样例，告诉我们如何使用。

[代码文件](./Docx/MQTT_Example.c)

### 1.2.4 拓展

使用[MQTTX](https://mqttx.app/zh/downloads)去模拟设备。

使用云产品流转，去将一个设备发送上来的消息转发到其他主题上。

### 1.2.5 参考资料

[参考资料1](https://mcxiaoke.gitbooks.io/mqtt-cn/content/mqtt/01-Introduction.html)

[参考资料2](https://www.emqx.com/zh/mqtt-guide)
