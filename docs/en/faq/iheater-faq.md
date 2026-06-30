# iHeater FAQ

---

### **Is iHeater compatible with my printer?**
iHeater is compatible with any printer running the **current version of Klipper**, and it can also work standalone with the **standalone firmware**.

---

### **What heater power is used?**

- The default heater is **200 W**.
- Other options: **100 W, 70 W, and 50 W** - mainly for **SLS printers** with small chambers where high temperatures are unnecessary.

---

### **Which controller is used in iHeater?**
It uses **STM32F042** (current version).

---

### **Which material should I use to print the enclosure?**
It is recommended to use **ABS/ASA** or higher temperature-resistant materials.

---

### **Will the enclosure melt at 130 °C?**

- The configured temperature is measured by the thermistor near the heater.
- At the aluminum tube edges, the temperature is lower due to airflow.
- **Spacer screws** are included to reduce heat load.

⚠️ On first startup, check operating limits:

- Raise the temperature stepwise (5-10°C per step).
- Probe the enclosure with a rigid tool.
- If plastic softens - reduce the set temperature.

---

### **What typical issues may occur?**
If the heater goes into **uncontrolled heating**, check the **thermistor installation**.
As with iDryer, an incorrectly installed thermistor results in overheating and enclosure damage.

---

### **How is safety handled?**
The safety system has multiple layers:

- **Software control**: automatic shutdown on overheating.
- **Watchdog timer**: MCU watchdog prevents hangs.
- **Hardware protection**: thermal fuse used. For testing - **KSD-9700** (auto-resets). For permanent use - **RH-135** (trips at 135 °C, permanently opens).
- A **fuse** is also present on the controller board for short-circuit protection.

---

### **What is the small blue board in the kit?**
It is a **220 V power supply** - test option. iHeater board supports its use.
⚠️ Only install if you are qualified and understand the risks.
If you connect this board, **never power via USB**. It is intended only for **standalone mode**.

---

### **What power supply is required?**
iHeater consumes about **100 mA**. No special requirements.

---

### **Can I power iHeater from the printer's internal power (110/220 V)?**
Yes. You can connect iHeater so that both printer and iHeater share the same switch.
Additionally, iHeater includes a special adapter that replaces **C7/C8 connectors**, allowing direct connection to screw terminals on the board.

---

### **Where can I find documentation?**
Documentation is available here:

- [Project GitHub](https://github.com/idryer)
- [docs.idryer.org](https://docs.idryer.org)
