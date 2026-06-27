# 组装

请阅读文档，下载并打印所需部件。开始组装前，请确认所有组件和工具都已准备齐全。

!!! danger "市电电压操作"
    所有连接 110–230 V 市电的操作都必须在设备断电状态下进行。更多信息请参见[安全](safety.md)章节。

## 组装前

建议先将整套系统**在桌面上**组装起来，不安装进外壳，并进行测试：

- 连接**所有**组件。
- 检查加热器、风扇、温度传感器是否正常工作。
- 将系统连接到 **Klipper**，或刷入 Standalone 固件，并确认其工作正常。

Video guide: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## 分步组装

### 安装控制板

![组装 iHeater](../img/iHeater_5484.jpg)

### 安装热敏电阻和 Thermal Protector

!!! warning "安装热敏电阻"
    请确保热敏电阻根部的裸露线段不会接触加热器的金属外壳。如有必要，请用 Kapton 胶带绝缘这些部位，或将其放入特氟龙管 / 热缩管中。

    加热器温度可能达到 140 °C。

!!! warning "安装 Thermal Protector"
    可以安装 KSD9700（Thermal Protector，自恢复型）或一次性 Thermal Fuse。

    KSD9700 会在过热时断开电路，并在冷却后自动重新闭合。Thermal Fuse（例如 **RH130**）触发后会永久断开电路，在故障情况下提供更可靠的保护。

    调试阶段使用 KSD9700，之后更换为 Thermal Fuse 以便长期使用。

![组装 iHeater](../img/iHeater_5489.jpg)
![组装 iHeater](../img/thermistor.jpg)

### 安装加热器

!!! warning "安装热敏电阻"
    将热敏电阻安装在加热器边缘，约位于散热片高度的中间位置。

    热敏电阻根部的裸露线段不得接触加热器的金属外壳。如有必要，请用 Kapton 胶带绝缘这些部位，或将其放入特氟龙管 / 热缩管中。

    加热器温度可能达到 140 °C。

![组装 iHeater](../img/iHeater_5491.jpg)

### 接线

![组装 iHeater](../img/iHeater_5494.jpg)

### 安装 НШВИ

![组装 iHeater](../img/iHeater_5496.jpg)

### 连接

![组装 iHeater](../img/iHeater_5498.jpg)

### 最终组装

![组装 iHeater](../img/iHeater_5500.jpg)

### 成品

![组装 iHeater](../img/iHeater_5506.jpg)
