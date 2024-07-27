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

在管理控制台下选择公共实例

点击创建产品

按要求输出，产品名称可以随意填写

然后返回，点击设备，创建设备  

创建完成后可以新建一个物模型变量

然后点击发布上线，然后就可以在功能定义这里看到刚刚定义的功能了。记得在Topic类列表中修改自定义user/get的权限改为订阅和发布，这样子我们才能通过这个Topic进行测试。

调试功能，在课上讲。然后就是配置menuconfig。

打开RW007

在软件包中找到阿里云的软件包

使能后需要修改里面的一些参数

在阿里云中找到这些参数并修改进去

然后再打开ENV末尾的使能样例，告诉我们如何使用。  

### 1.2.4 拓展

使用[MQTTX](https://mqttx.app/zh/downloads)去模拟设备。

使用云产品流转，去将一个设备发送上来的消息转发到其他主题上。

### 1.2.5 参考资料

[参考资料1](https://mcxiaoke.gitbooks.io/mqtt-cn/content/mqtt/01-Introduction.html)

[参考资料2](https://www.emqx.com/zh/mqtt-guide)  
# 二、组件（Component）

定义：指的是一个可以独立开发、测试、部署和维护的软件单元。

## 2.1 文件系统

在本节中，我们会基于板载的W25Q64来学习如何使用[文件系统](https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/programming-manual/filesystem/filesystem)

### 2.1.1 文件系统定义

**DFS** 是 RT-Thread 提供的虚拟文件系统组件，全称为 Device File System。

### 2.1.2 文件系统架构

在 RT-Thread DFS 中，文件系统有统一的根目录，使用 `/` 来表示。

### 2.1.3 文件系统种类

| 类型  |                             特点                             |
| ----- | :----------------------------------------------------------: |
| FatFS | FatFS 是专为小型嵌入式设备开发的一个兼容微软 FAT 格式的文件系统，采用ANSI C编写，具有良好的硬件无关性以及可移植性，是 RT-Thread 中最常用的文件系统类型。我们今天使用到的elm_fat就是这个类型。 |
| RomFS | 传统型的 RomFS 文件系统是一种简单的、紧凑的、只读的文件系统，不支持动态擦写保存，按顺序存放数据，因而支持应用程序以 XIP(execute In Place，片内运行) 方式运行，在系统运行时, 节省 RAM 空间。我们一般拿其作为挂载根目录的文件系统 |
| DevFS | 即设备文件系统，在 RT-Thread 操作系统中开启该功能后，可以将系统中的设备在 `/dev` 文件夹下虚拟成文件，使得设备可以按照文件的操作方式使用 read、write 等接口进行操作。 |
| UFFS  | UFFS 是 Ultra-low-cost Flash File System（超低功耗的闪存文件系统）的简称。它是国人开发的、专为嵌入式设备等小内存环境中使用 Nand Flash 的开源文件系统。与嵌入式中常使用的 Yaffs 文件系统相比具有资源占用少、启动速度快、免费等优势。 |
| NFS   | NFS 网络文件系统（Network File System）是一项在不同机器、不同操作系统之间通过网络共享文件的技术。在操作系统的开发调试阶段，可以利用该技术在主机上建立基于 NFS 的根文件系统，挂载到嵌入式设备上，可以很方便地修改根文件系统的内容。 |

### 2.1.4 POSIX接口层

POSIX 表示可移植操作系统接口（Portable Operating System Interface of UNIX，缩写 POSIX），POSIX 标准定义了操作系统应该为应用程序提供的接口标准，是 IEEE 为要在各种 UNIX 操作系统上运行的软件而定义的一系列 API 标准的总称。

> 文件描述符：`file descriptor`（fd），每一个文件描述符会与一个打开文件相对应，同时，不同的文件描述符也可能指向同一个文件。可以简单理解为它可以帮助我们找到我们需要的文件。

在文件系统中它提供了四个重要的接口：

![文件管理常用函数](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-standard/programming-manual/filesystem/figures/fs-mg.png)

还有一些其他常用的API：

| API                                             | 描述         |
| ----------------------------------------------- | ------------ |
| int `rename`(const char *old, const char *new); | 文件重命名   |
| int `stat`(const char *file, struct stat *buf); | 获取文件状态 |
| int `unlink`(const char *pathname);             | 删除文件     |

### 2.1.5 目录管理

除了文件的管理之外我们还需要对目录进行管理，管理使用的API：

![目录管理常用函数](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-standard/programming-manual/filesystem/figures/fs-dir-mg.png)

### 2.1.6 文件系统启动流程

#### 2.1.6.1 DFS组件初始化

在这个环节中我们会初始化文件系统注册表、有关文件描述符的各种类型的锁。如果使用了设备文件系统的化还会对其做一定的初始化。

#### 2.1.6.2 各文件系统注册到DFS中

各自文件系统调用自己的一个init函数注册到DFS中。就是将上面所将的FatFS，RomFS...注册到DFS中等待使用。

#### 2.1.6.3 挂载根目录

使用RomFS创建一个简单的`”/“`根目录，使用到的其他文件系统需要挂载到这个根目录下做拓展。

## 2.2 FAL（搭配SFUD驱动使用）

### 2.2.1 SFUD

**[SFUD](https://github.com/armink/SFUD)**：(Serial Flash Universal Driver) 串行 Flash 通用驱动库。由于现有市面的串行 Flash 种类居多，各个 Flash 的规格及命令存在差异， SFUD 就是为了解决这些 Flash 的差异现状而设计。其实就是帮我们把底层的驱动代码写好，将设备抽象成SPI_FLASH，并提供给我们类似：`sfud_get_device（）`、`sfud_read（）`、`sfud_erase（）`、`sfud_write（）`...等函数接口帮助我们能够实现对不同Flash的读写。

### 2.2.2 FAL

**Fal组件**：(Flash Abstraction Layer) Flash 抽象层，是对 Flash 及基于 Flash 的分区进行管理、操作的抽象层，对上层统一了 Flash 及 分区操作的 API。并具有以下特性：

- 支持静态可配置的分区表，并可关联多个 Flash 设备；
- 分区表支持 **自动装载** 。避免在多固件项目，分区表被多次定义的问题；
- 代码精简，对操作系统 **无依赖** ，可运行于裸机平台，比如对资源有一定要求的 Bootloader；
- 统一的操作接口。保证了文件系统、OTA、NVM（例如：[EasyFlash](https://github.com/armink-rtt-pkgs/EasyFlash)） 等对 Flash 有一定依赖的组件，底层 Flash 驱动的可重用性；
- 自带基于 Finsh/MSH 的测试命令，可以通过 Shell 按字节寻址的方式操作（读写擦） Flash 或分区，方便开发者进行调试、测试；

![FAL framework](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-standard/programming-manual/fal/figures/fal_framework.png)

### 2.2.3 FAL API

![FAL API](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-standard/programming-manual/fal/figures/fal-api.png)

### 2.2.4 FAL初始化流程



## 2.3 文件系统结合FAL配置W25Q64

### 2.3.1 样例

W25Q64

首先会在`rt_hw_spi_flash()`（在INIT_COMPONENT_EXPORT）中会把一个`spi20`的spi设备挂载在`spi2`总线上，然后通过`rt_sfud_flash_probe`将这个`spi20`设备跟一个`SPI_FLASH`设备（命名为`W25Q64`）进行绑定。然后在FAL中对这个`SPI_FLASH（W25Q64）`设备进行分区，然后对相应的区创建`BLK设备`。

接着我们对这个`BLK设备`进行格式化（即挑选一种文件系统去管理这个BLK设备），然后将这个格式化好的文件系统进行挂载（分配到对应路径上）这样我们就可以使用组件的`API`对`W25Q64`进行读写了。这里使用了POSIX协议接口，我们只需要使用`open()`、`close()`、`read()`、`write()`就可以完成对文件的读写。也可以用`mkdir()`、`opendir()`、`readdir()`、`closedir()`来对目录进行管理。


1. 开启板上外设


2. 配置自动挂载


3. 配置Component组件


4. 配置DFS


5. 配置elmFat

## 最后，附上源码  
```
#include <board.h>
#include <rtthread.h>
#include <drv_gpio.h>
#include <dfs_posix.h>//需要添加软件包进这里

//定义要写入的内容
char String[] = "Hello, RT-Thread.Welcom to RSOC!";

//定义接受文件内容的缓冲区
char buffer[100] = {};

void FileSystem_Test(void *parameter)
{
    //文件描述符
    int fd;

    //用只写方式打开文件,如果没有该文件,则创建一个文件
    fd = open("/fal/FileTest.txt", O_WRONLY | O_CREAT);

    //如果打开成功
    if (fd >= 0)
    {
        //写入文件
        write(fd, String, sizeof(String));

        rt_kprintf("Write done.\n");

        //关闭文件
        close(fd);
    }
    else
    {
        rt_kprintf("File Open Fail.\n");
    }

    //用只读方式打开文件
    fd = open("/fal/FileTest.txt", O_RDONLY);

    if (fd>= 0)
    {
        //读取文件内容
        rt_uint32_t size = read(fd, buffer, sizeof(buffer));
    
        if (size < 0)
        {
            rt_kprintf("Read File Fail.\n");
            return ;
        }

        //输出文件内容
        rt_kprintf("Read from file test.txt : %s \n", buffer);

        //关闭文件
        close(fd);
    }
    else
    {
        rt_kprintf("File Open Fail.\n");
    }
}
//导出命令
MSH_CMD_EXPORT(FileSystem_Test, FileSystem_Test);

static void readdir_sample(void)
{
    DIR *dirp;
    struct dirent *d;

    /* 打开 / dir_test 目录 */
    dirp = opendir("/fal");
    if (dirp == RT_NULL)
    {
        rt_kprintf("open directory error!\n");
    }
    else
    {
        /* 读取目录 */
        while ((d = readdir(dirp)) != RT_NULL)
        {
            rt_kprintf("found %s\n", d->d_name);
        }

        /* 关闭目录 */
        closedir(dirp);
    }
}
/* 导出到 msh 命令列表中 */
MSH_CMD_EXPORT(readdir_sample, readdir sample);

/*
#define WIFI_CS GET_PIN(F, 10)
void WIFI_CS_PULL_DOWM(void)
{
    rt_pin_mode(WIFI_CS, PIN_MODE_OUTPUT);
    rt_pin_write(WIFI_CS, PIN_LOW);
}
INIT_BOARD_EXPORT(WIFI_CS GET_PIN);
*/
```
## mqtt-example.c  
```
/*
 * Copyright (C) 2015-2018 Alibaba Group Holding Limited
 * 
 * Again edit by rt-thread group
 * Change Logs:
 * Date          Author          Notes
 * 2019-07-21    MurphyZhao      first edit
 */

#include "rtthread.h"
#include "dev_sign_api.h"
#include "mqtt_api.h"

char DEMO_PRODUCT_KEY[IOTX_PRODUCT_KEY_LEN + 1] = {0};
char DEMO_DEVICE_NAME[IOTX_DEVICE_NAME_LEN + 1] = {0};
char DEMO_DEVICE_SECRET[IOTX_DEVICE_SECRET_LEN + 1] = {0};

void *HAL_Malloc(uint32_t size);
void HAL_Free(void *ptr);
void HAL_Printf(const char *fmt, ...);
int HAL_GetProductKey(char product_key[IOTX_PRODUCT_KEY_LEN + 1]);
int HAL_GetDeviceName(char device_name[IOTX_DEVICE_NAME_LEN + 1]);
int HAL_GetDeviceSecret(char device_secret[IOTX_DEVICE_SECRET_LEN]);
uint64_t HAL_UptimeMs(void);
int HAL_Snprintf(char *str, const int len, const char *fmt, ...);

#define EXAMPLE_TRACE(fmt, ...)  \
    do { \
        HAL_Printf("%s|%03d :: ", __func__, __LINE__); \
        HAL_Printf(fmt, ##__VA_ARGS__); \
        HAL_Printf("%s", "\r\n"); \
    } while(0)

static void example_message_arrive(void *pcontext, void *pclient, iotx_mqtt_event_msg_pt msg)
{
    iotx_mqtt_topic_info_t     *topic_info = (iotx_mqtt_topic_info_pt) msg->msg;

    switch (msg->event_type) {
        case IOTX_MQTT_EVENT_PUBLISH_RECEIVED:
            /* print topic name and topic message */
            EXAMPLE_TRACE("Message Arrived:");
            EXAMPLE_TRACE("Topic  : %.*s", topic_info->topic_len, topic_info->ptopic);
            EXAMPLE_TRACE("Payload: %.*s", topic_info->payload_len, topic_info->payload);
            EXAMPLE_TRACE("\n");
            break;
        default:
            break;
    }
}

static int example_subscribe(void *handle)
{
    int res = 0;
    const char *fmt = "/%s/%s/user/get";
    char *topic = NULL;
    int topic_len = 0;

    topic_len = strlen(fmt) + strlen(DEMO_PRODUCT_KEY) + strlen(DEMO_DEVICE_NAME) + 1;
    topic = HAL_Malloc(topic_len);
    if (topic == NULL) {
        EXAMPLE_TRACE("memory not enough");
        return -1;
    }
    memset(topic, 0, topic_len);
    HAL_Snprintf(topic, topic_len, fmt, DEMO_PRODUCT_KEY, DEMO_DEVICE_NAME);

    res = IOT_MQTT_Subscribe(handle, topic, IOTX_MQTT_QOS0, example_message_arrive, NULL);
    if (res < 0) {
        EXAMPLE_TRACE("subscribe failed");
        HAL_Free(topic);
        return -1;
    }

    HAL_Free(topic);
    return 0;
}

static int example_publish(void *handle)
{
    int             res = 0;
    const char     *fmt = "/%s/%s/user/get";
    char           *topic = NULL;
    int             topic_len = 0;
    char           *payload = "{\"message\":\"hello!\"}";

    topic_len = strlen(fmt) + strlen(DEMO_PRODUCT_KEY) + strlen(DEMO_DEVICE_NAME) + 1;
    topic = HAL_Malloc(topic_len);
    if (topic == NULL) {
        EXAMPLE_TRACE("memory not enough");
        return -1;
    }
    memset(topic, 0, topic_len);
    HAL_Snprintf(topic, topic_len, fmt, DEMO_PRODUCT_KEY, DEMO_DEVICE_NAME);

    res = IOT_MQTT_Publish_Simple(0, topic, IOTX_MQTT_QOS0, payload, strlen(payload));
    if (res < 0) {
        EXAMPLE_TRACE("publish failed, res = %d", res);
        HAL_Free(topic);
        return -1;
    }

    HAL_Free(topic);
    return 0;
}

static void example_event_handle(void *pcontext, void *pclient, iotx_mqtt_event_msg_pt msg)
{
    EXAMPLE_TRACE("msg->event_type : %d", msg->event_type);
}

/*
 *  NOTE: About demo topic of /${productKey}/${deviceName}/user/get
 *
 *  The demo device has been configured in IoT console (https://iot.console.aliyun.com)
 *  so that its /${productKey}/${deviceName}/user/get can both be subscribed and published
 *
 *  We design this to completely demonstrate publish & subscribe process, in this way
 *  MQTT client can receive original packet sent by itself
 *
 *  For new devices created by yourself, pub/sub privilege also requires being granted
 *  to its /${productKey}/${deviceName}/user/get for successfully running whole example
 */

static int mqtt_example_main(int argc, char *argv[])
{
    void                   *pclient = NULL;
    int                     res = 0;
    int                     loop_cnt = 0;
    iotx_mqtt_param_t       mqtt_params;

    HAL_GetProductKey(DEMO_PRODUCT_KEY);
    HAL_GetDeviceName(DEMO_DEVICE_NAME);
    HAL_GetDeviceSecret(DEMO_DEVICE_SECRET);

    EXAMPLE_TRACE("mqtt example");

    /* Initialize MQTT parameter */
    /*
     * Note:
     *
     * If you did NOT set value for members of mqtt_params, SDK will use their default values
     * If you wish to customize some parameter, just un-comment value assigning expressions below
     *
     **/
    memset(&mqtt_params, 0x0, sizeof(mqtt_params));

    /**
     *
     *  MQTT connect hostname string
     *
     *  MQTT server's hostname can be customized here
     *
     *  default value is ${productKey}.iot-as-mqtt.cn-shanghai.aliyuncs.com
     */
    /* mqtt_params.host = "something.iot-as-mqtt.cn-shanghai.aliyuncs.com"; */

    /**
     *
     *  MQTT connect port number
     *
     *  TCP/TLS port which can be 443 or 1883 or 80 or etc, you can customize it here
     *
     *  default value is 1883 in TCP case, and 443 in TLS case
     */
    /* mqtt_params.port = 1883; */

    /**
     *
     * MQTT request timeout interval
     *
     * MQTT message request timeout for waiting ACK in MQTT Protocol
     *
     * default value is 2000ms.
     */
    /* mqtt_params.request_timeout_ms = 2000; */

    /**
     *
     * MQTT clean session flag
     *
     * If CleanSession is set to 0, the Server MUST resume communications with the Client based on state from
     * the current Session (as identified by the Client identifier).
     *
     * If CleanSession is set to 1, the Client and Server MUST discard any previous Session and Start a new one.
     *
     * default value is 0.
     */
    /* mqtt_params.clean_session = 0; */

    /**
     *
     * MQTT keepAlive interval
     *
     * KeepAlive is the maximum time interval that is permitted to elapse between the point at which
     * the Client finishes transmitting one Control Packet and the point it starts sending the next.
     *
     * default value is 60000.
     */
    /* mqtt_params.keepalive_interval_ms = 60000; */

    /**
     *
     * MQTT write buffer size
     *
     * Write buffer is allocated to place upstream MQTT messages, MQTT client will be limitted
     * to send packet no longer than this to Cloud
     *
     * default value is 1024.
     *
     */
    /* mqtt_params.write_buf_size = 1024; */

    /**
     *
     * MQTT read buffer size
     *
     * Write buffer is allocated to place downstream MQTT messages, MQTT client will be limitted
     * to recv packet no longer than this from Cloud
     *
     * default value is 1024.
     *
     */
    /* mqtt_params.read_buf_size = 1024; */

    /**
     *
     * MQTT event callback function
     *
     * Event callback function will be called by SDK when it want to notify user what is happening inside itself
     *
     * default value is NULL, which means PUB/SUB event won't be exposed.
     *
     */
    mqtt_params.handle_event.h_fp = example_event_handle;

    pclient = IOT_MQTT_Construct(&mqtt_params);
    if (NULL == pclient) {
        EXAMPLE_TRACE("MQTT construct failed");
        return -1;
    }

    res = example_subscribe(pclient);
    if (res < 0) {
        IOT_MQTT_Destroy(&pclient);
        return -1;
    }

    while (1) {
        if (0 == loop_cnt % 20) {
            example_publish(pclient);
        }

        IOT_MQTT_Yield(pclient, 200);

        loop_cnt += 1;
    }

    return 0;
}
#ifdef FINSH_USING_MSH
MSH_CMD_EXPORT_ALIAS(mqtt_example_main, ali_mqtt_sample, ali coap sample);
#endif
```
