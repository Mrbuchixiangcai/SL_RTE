# SL_RTE
SL_RTE是多个项目开发实践中积累下来的一些代码的整合，主要包含八个部分：Core层，即脱离于硬件平台的部分；BSP层，即适配不同硬件平台的部分；Board层，即针对具体的板级的支持包；Utils层，即第三方库；Port层，移植RTE需要变动的文件；GUI层，以lvgl为内核的GUI；MV层，以openmv为内核的机器视觉模块；Module层，对以上几层的进一步封装，为用户使用SL_RTE提供一个较为规范的引导。
# 使用说明
使用SL_RTE，可以大大减少嵌入式开发尤其是STM32系列开发的时间（因为BSP层和Board层大都是STM32系列的实现，如果用户本身积累了相当的MCU相关的BSP，那么同样可以用于SL_RTE）。
实时操作系统下的SL_RTE和安卓的机制类似，设计好的GUI界面的刷新只在GUI线程中完成，其余线程不能对GUI界面产生直接影响。
在3.7.0319版本中，正式完成了APP_Module，该模块作为RTE_Module部分的第一个模块，初步确立了基于SL_RTE的应用编写规范。
（1）一个完整的应用可以由以下几种方式实现：
裸机；
实时操作系统；
裸机+GUI；
实时操作系统+GUI；
（2）在使用实时操作系统但不使用GUI的环境下，可以采用线程+状态机的方式实现整个应用；在裸机且不使用GUI的环境下，可以采用前台（时间片轮询+状态机）+后台（各种中断）的方式实现整个应用。
（3）在使用GUI时，一个完成的应用可以由一个或者多个图形APP构成。如果是在操作系统环境下，每个APP可以拥有一个线程。对于拥有线程的APP而言，其app_run和app_close需要实现线程的创建和删除。
（4）每个APP的生命周期由如下几个部分组成：
a.建立APP初期： （该部分由RTE自己完成 不需要用户交互参与）
APP建立->APP_GUI绘制->app_sc_open
该部分完成于整个SL_RTE初始化期间，在GUI线程进入线程体之前。
b.APP开始：（该部分需要用户交互参与）
用户点击APP图标->app_run->APP窗口绘制->app_win_open
该部分在功能上实现了APP的主窗体绘制以及APP内部窗体绘制，同时在app_run函数中，应当完成APP线程的建立。
c.APP运行时操作：（该部分需要用户交互参与）
用户点击关闭->关闭动画->app_win_close->APP窗口销毁
此时APP真正关闭 线程退出 APP内部相关资源需要释放
用户点击最小化>APP窗口隐藏
此时APP进入后台状态 线程继续运行
d.APP生命周期结束：（该部分通常不需要实现）
调用APP_Remove->销毁可能存在的APP窗口->APP_GUI销毁->释放APP资源
这一阶段是真正意义上的APP生命周期结束，如果发生，那么主界面上的APP相关GUI也会消失，所有APP有关的资源都会被释放。
（5）对于拥有线程的APP而言，线程内可以由状态机模块实现，不同的状态可以内部切换，也可以由用户与GUI交互实现切换（即GUI线程对APP线程的改变）。
（6）在裸机环境下，GUI模块作为RR的system组的一个定时器存在。
# 主要内容
RTE框架如下：

-RTE

--RTE_APP
---包含内存管理、类shell交互、不同Core的文件、一个环形队列、一个状态机实现、一个软件定时器、字符串处理库等内容。

--RTE_BSP
---包含任何Chip必有的三个部分：按键、串口、LED以方便用户对运行于SL_RTE下的代码进行调试。

--RTE_Board
---包含常用的板级支持包。

---STM32F103：ADC、BH1750、CRC、DHT11、E2PROM、ESP8266、片内Flash模拟E2PROM、GSMA9、I2C、LCD、PM25、PWM
              RC522、IO模拟串口、SPI、SR501、GPIO。  
              
   STM32F407：类似F1。
   
   IOTL475：移植官方驱动。
   
   STM32F767:待上传。
   
   STM32H743：待上传。

--RTE_Config
---可在KEIL中进行图形化配置的RTE配置文件。

--RTE_Example
---KEIL环境下不同开发板的DEMO。

--RTE_GUI
---本RTE自带的GUI，以LVGL为内核；

--RTE_MV
---本RTE自带的机器视觉库，以OPENMV为内核；

--RTE_OldVersion
---老版本。

--RTE_Port
---移植到不同平台所需文件。

--RTE_Utils
---一些第三方库。

# 版本历史

2017/09/22  新版本1.0release。

2017/09/23  完全剥离bsp和app 确保app使用时独立于硬件底层（SoftTimer除外 但针对coterxm内核mcu一致）；
            加入bget内存管理 将部分app变为动态内存分配机制（用于debug的串口缓存）；
            修改ringbuffer 使出列和入列变为动态内存分配。
            
2017/11/06  结合tmlib开始2.0版本编写 基于F7以及HAL库。

2017/11/12  2.0版本的除了spi其他都已经测试通过；
            为了配合mppt项目开始完成针对f1系列以及固件库的适配；
            修改app中led和key到bsp中去。
            
2017/11/13  完成com key led对于f1的适配；
            修改了key和led使其适配于m3和m4系列内核；
            单独为m3内核建立comf1；
            确定新bsp撰写规范；
            攥写了bsp_timerbase。
            
2018/04/17  新版本开始编写 3.0；
            app_mem 更换rtx自带的内存管理系统做为底层 方便管理多块内存空间；
            增加app_stdio 替换printf；
            更换APP_Config和BSP_Config为适用于MDK开发环境，可勾选配置。
            
2018/05/30  基本完成F1板级支持库的开发；
            开始做F4的板级支持库和bsp包；
            
2018/06/15  完成对bget的重写，使其支持多块内存，删除app_mem；
            重命名SL_LIB为SL-RTE；
            修改F1的串口接收为DMA工作模式；
            优化对MDK开发的适配；
            最新的RTE在STM32F103C8T平台上通过1ms循环，debug指令暴力测试；
            
2018/06/23  决定开源，上传到github。

2018/08/07  版本号2.0.0807
            更换命名；
            重写时间片；
            重写Debug并更名为shell；
            重写Config文件；
            管理思想改为动态管理；

2018/08/26  版本号2.1.0826
            整理了RTE文件框架；
            移植了ESAYFLASH作为KEY-VALUE数据库；
            目前用于TM1294项目（SHELL与SM静态版本）；

2018/08/31  版本号2.2.0831
            换用新的内存管理；
            重写了roundrobin和shell；
            目前用于407遥控开关项目；（SHELL与SM动静态结合版本，纯动态版本会产生较多的内存碎片）；

2018/09/02  版本号2.2.0902
            407的串口bsp文件重写；
            完成RTE_Stdlib和RTE_Stdio的移植；
            完善RTE_RetargetIO，使其适配于非MicroLib环境；
            修正了Config文件；

2018/09/30  版本号2.3.0930
            增加RTE_Vec和RTE_List；
            利用RTE_Vec重新对Shell、Timer、SM等模块进行重构；
            开始移植GUI；

2018/10/2   版本号3.0.1002
            完成了GUI的移植，GUI版本号为5.0.1；
            完善了RR部分代码；

2018/11/2   版本号3.1.1102
            重构GUI代码，GUI版本号为5.2.0；
            重构目录树；
            重写RTE_Config；

2018/12/27  版本号3.2.1227
            lvgl内核升级到5.2.1，修复部分RTE_APP；
            一些其他小改动；

2019/01/07  版本号3.3.0107
            lvgl更新优化接口；
            MEM、Message增加互斥锁；
            STDOUT和KVDB优化互斥锁逻辑；
            LOG用于各个模块；
            升级日志模块，可选输出到RTE_Stdout、FILE、Flash；
            优化include结构；
2019/01/13  版本号3.3.0113
            修复了MEM、Message的互斥锁逻辑；
2019/01/24  版本号3.3.0124
            删除无用的GUI_File模块；
            完善PC使用SL_RTE的逻辑；
            更新了Shell模块，并为GUI增加了Shell支持，当前包括obj建立和删除。
2019/01/30  版本号3.3.0130
            完善PC使用SL_RTE的逻辑；
            为GUI增加了动态管理；
2019/03/13  版本号3.4.0313
            升级GUI到5.3.0
            升级KVDB到4.0.0
2019/03/16  版本号3.6.0316
            换用umm_malloc作为内存管理模块；
2019/03/17  版本号3.7.0317
            换用tlsf作为内存管理模块；
2019/03/19  版本号3.8.0319
            完成了APP_Module的编写，正式确立GUI和RTOS编程规范；

下一版本计划：
1、IAP库的开发和移植；
2、lua脚本解释器；
3、RTE_Simulator的功能完善：网络、串口通讯、文件、线程机制转换到cmsis接口；

# 其他
请参见RTE参考手册。
