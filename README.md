# 独立的硬件设备驱动开发

## 第一周

独立的硬件设备驱动开发，需要用到硬件开发板厂商用到的文档及代码等资源。

可供参考的Linux驱动和SDK驱动及技术文档如下：

* 飞腾派相关开发文档：
https://github.com/elliott10/dev-hw-driver/tree/main/phytiumpi/docs

* ArceOS for 飞腾派驱动开发参考案例：
https://github.com/arceos-org/arceos/pull/222

* 飞腾派Linux代码
https://gitee.com/phytium_embedded/phytium-linux-kernel

* [飞腾派Linux用户使用开发手册](https://gitee.com/phytium_embedded/phytium-embedded-docs/blob/master/phytiumpi/linux/%E9%A3%9E%E8%85%BE%E6%B4%BEOS%E7%94%A8%E6%88%B7%E4%BD%BF%E7%94%A8%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%20v2.1.pdf)

* 飞腾派SDK代码
https://gitee.com/phytium_embedded/phytium-standalone-sdk

* [飞腾派裸机SDK指导手册 （含各类驱动示例）](https://gitee.com/phytium_embedded/phytium-embedded-docs/blob/master/phytiumpi/rtos/%E9%A3%9E%E8%85%BE%E6%B4%BE%E8%A3%B8%E6%9C%BASDK%E6%8C%87%E5%AF%BC%E6%89%8B%E5%86%8C-v1.0.pdf)


### 作业

#### 作业2

* 基于模拟器Qemu，其ARM64 virt型号提供了GPIO来实现关机功能。

我们需要编写具有关机功能的GPIO驱动，最终实现在ArceOS上关机。

**Qemu GPIO驱动关机原理**

当我们按下GPIO关机按钮时，这里是在Qemu命令终端输入 system_powerdown命令来触发，
GPIORIS（中断状态寄存器PrimeCell GPIO raw interrupt status）中的第三位将从0跳变到1。
而当GPIOIE（中断掩码寄存器PrimeCell GPIO interrupt mask）中的第三位为1时，GPIO处理芯片将向GIC中断控制器发送一次中断，中断号为39。
OS收到中断后，需要对此次GPIO中断进行清除，将GPIOIC（中断清除寄存器PrimeCell GPIO interrupt clear）的对应位置为1
然后OS进行关机操作。

**初始化关机的GPIO中断**
1. 设置寄存器GPIO_IRQ为ICFGR_LEVEL，设置电平触发；
2. 设置寄存器GPIO_IRQ为0，设定优先级；
3. 若多核则需要设置中断目标核；
4. 清空中断请求，寄存器GPIO_IRQ
5. 使能中断，寄存器GPIO_IRQ
6. 使能GPIO的poweroff key中断，启用pl061 gpio中的3号线中断，设置GPIOIE::IO3::Enabled

**对关机GPIO中断进行处理**
1. Qemu命令终端输入 system_powerdown 来触发关机按钮
2. 中断处理函数中，处理39号中断，通过读取pl061来清除它的中断信号
3. 执行关机指令：`mov w0, #0x18`; `hlt #0xF000`

#### 作业1

若有硬件板子的童鞋，可以继续选择原作业1：

ArceOS for 飞腾派硬件平台上实现GPIO点灯驱动

## 第二周

作业：
