# iHeater Connection Issues and How to Fix Them

When using **iHeater**, connection stability issues may sometimes occur (disconnects, "loss" of the MCU, unstable operation).  
In most cases, this is caused not by the device itself, but by external factors such as vibration, electromagnetic interference, or load-related characteristics.

Below are the main causes and ways to resolve them.

---

## 1. USB Cable Vibration

!!! warning "Symptoms"
    - Intermittent connection drops  
    - The device disappears from the system  
    - The connection is restored when the cable is touched  

!!! info "Cause"
    Vibrations from the printer can cause tiny movements of the USB connector, leading to brief contact loss.

!!! success "Solution"
    - Secure the USB cable firmly in the connector  
    - Eliminate any tension on the cable  
    - If necessary:
        - use a cable with a tighter fit  
        - secure the cable with hot glue / a zip tie / a holder  

---

## 2. Interference from Power Wires

!!! warning "Symptoms"
    - Loss of connection when heating or the fan is turned on  
    - Random device restarts  
    - Unstable operation without an obvious reason  

!!! info "Cause"
    AC power wires generate electromagnetic interference that can be induced into the USB cable.

 ![ferrite bead](../../img/ferrite_bead.png)

!!! success "Solution"
    - Keep the USB cable and power wires as far apart as possible  
    - Do not route them through the same cable channel  
    - Avoid running them in parallel over long distances  
    - Install a ferrite filter (ferrite bead) on the USB cable closer to the controller and/or the printer board

---

## 3. Fan Interference

!!! warning "Symptoms"
    - Loss of connection when the fan is turned on or off  
    - Errors that coincide with cooler operation  
    - Instability during PWM control  

!!! info "Cause"
    A 110-220 V fan uses a switching power supply, which can generate interference similar to any other switching power supply.
    This interference can affect signal lines.

![ferrite bead](../../img/snubber1.png)
![ferrite bead](../../img/snubber2.png)

!!! success "Solution"
    It is recommended to install an **RC snubber** in parallel with the fan, or use a ferrite filter.

    
