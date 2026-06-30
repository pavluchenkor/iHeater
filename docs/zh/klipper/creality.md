## 通过 Creality Helper Script 为 Creality 打印机安装 iHeater 固件

为确保 iHeater 固件安装和集成顺利完成，请按照以下分步说明操作：

### 1. 安装 Creality Helper Script

前往 Creality Helper Script 项目的文档页面，并按照脚本安装说明操作。

**资源：**

* 视频指南：[YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* 文字说明：[guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. 获取打印机的 root 访问权限和文件系统访问权限

该脚本会开放 Mainsail 访问权限，以及固件配置文件的访问权限。安装成功后，请确认可以通过浏览器进入打印机界面，并访问配置文件。

### 3. 删除旧的 `fan-control.cfg`

在使用 Helper Script 的 Creality 打印机上，默认可能已经创建了包含 `M141` 和 `M191` 宏的 `fan-control.cfg` 文件。它会与 iHeater 配置中的同类宏发生冲突。

将文件重命名：

```
/usr/data/printer_data/config/fan-control.cfg
```
为 fan-control.cfg.bak

### 4. 复制新的 `fan-control.cfg`

将其替换为与腔体温度控制宏和逻辑兼容的 [fan-control.cfg](../../../printers/creality/config/fans-control.cfg) 版本。

将新文件放入同一文件夹：

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. 添加 iHeater 配置

将 `iheater.cfg` 文件复制到同一目录：

```
/usr/data/printer_data/config/iheater.cfg
```

然后打开 `printer.cfg`，并在文件末尾添加以下行：

```ini
[include iheater.cfg]
```

---

接下来，请按照 iHeater 设置说明继续操作：配置热敏电阻、加热器、工作模式和宏。

!!! warning "如果无法在打印机上构建并刷写固件"
    [请参阅 WSL 章节](https://github.com/pavluchenkor/iHeater/tree/main/User-mods/software/WSL2_Ubuntu_FF)
