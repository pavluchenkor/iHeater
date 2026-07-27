# 关于 iHeater 项目

iHeater 是一款紧凑型加热器，用于在 3D 打印机中创建主动恒温腔。它特别适用于带有封闭式或专有电子系统的机型，如 Creality、Bambu Lab、FlashForge，这些设备通常没有可用于连接加热器、风扇和热敏电阻的空闲接口。

它通过 USB 连接，并独立于主板限制运行。根据固件不同，它可以与 Klipper 完全集成，也可以独立工作。

结合热床加热，iHeater 可确保腔体均匀升温，这是打印 ABS、PA、PC 和其他工程塑料时的关键因素。设备会根据空气温度动态控制加热，在腔体内部创建稳定条件，避免过热和温度波动。

提供两个版本：

- 100 W — 适用于小型打印机（存档版）
- 200 W — 适用于更大尺寸的打印机

![iHeater](../img/iHeater_promo.png)

[您可以使用此计算器进行初步估算](https://docs.google.com/spreadsheets/d/1u6XrWLFZGOUnRlFPjjGsJB_GLuFFIs3fFWLCp2-K8gc/edit?usp=sharing)

## 使用方式

### 由 Klipper 控制

该板在 Klipper 中作为独立 MCU 工作，完全自主地控制腔体加热和风扇。由 220 V 供电不会增加打印机电源负载，因为原厂电源通常已接近负载极限。

![PCB](../img/iHeater_200_PCB.png)

该板的成本与基于微控制器、固态继电器和必要组件自行组装类似方案相当，甚至更低。对于爱好者，仍然可以自行组装类似方案。

### 使用 iHeater 固件

iHeater 板是自足的，包含作为独立设备使用所需的全部外围电路。目标温度通过连续按下 MODE 按钮设置，并由三个 LED 显示。

## 许可证

本项目基于 MIT 许可证发布。详情见 [LICENSE](license.md) 文件。

!!! danger "使用加热元件"
    使用加热元件和温度控制存在起火和设备损坏风险。请遵守安全预防措施。更多信息见 [安全](safety.md) 章节。
