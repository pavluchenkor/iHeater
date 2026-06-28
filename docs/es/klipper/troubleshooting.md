# Problemas de comunicación de iHeater y cómo resolverlos

Al usar **iHeater**, en algunos casos pueden aparecer problemas de estabilidad de la conexión (desconexiones, "pérdida" de la MCU, funcionamiento inestable).  
En la mayoría de los casos, esto no está relacionado con el propio dispositivo, sino con factores externos: vibraciones, interferencias electromagnéticas o características de la carga.

A continuación se indican las causas principales y cómo solucionarlas.

---

## 1. Vibración del cable USB

!!! warning "Síntomas"
    - Desconexiones periódicas  
    - El dispositivo "desaparece" del sistema  
    - La comunicación se restablece al tocar el cable  

!!! info "Causa"
    Las vibraciones de la impresora pueden provocar micromovimientos del conector USB, lo que causa una pérdida breve de contacto.

!!! success "Solución"
    - Fije firmemente el cable USB en el conector  
    - Elimine la tensión del cable  
    - Si es necesario:
        - use un cable con un ajuste más firme  
        - fije el cable con termocola / una brida / un soporte  

---

## 2. Interferencias de los cables de potencia

!!! warning "Síntomas"
    - Pérdida de comunicación al encender la calefacción o el ventilador  
    - Reinicios aleatorios del dispositivo  
    - Funcionamiento inestable sin una causa evidente  

!!! info "Causa"
    Los cables de alimentación de corriente alterna generan interferencias electromagnéticas que se inducen en el cable USB.

 ![ferrite bead](../../img/ferrite_bead.png)

!!! success "Solución"
    - Separe el cable USB y los cables de potencia tanto como sea posible  
    - No los coloque en el mismo canal de cables  
    - Evite tendidos paralelos en tramos largos  
    - Instale un filtro de ferrita (cilindro de ferrita) en el cable USB, más cerca del controlador y/o de la placa de la impresora

---

## 3. Interferencias del ventilador

!!! warning "Síntomas"
    - Pérdida de comunicación al encender/apagar el ventilador  
    - Fallos que coinciden con el funcionamiento del ventilador  
    - Inestabilidad con control PWM  

!!! info "Causa"
    El ventilador de 110-220 V está equipado con una fuente de alimentación conmutada y puede generar interferencias similares a las de cualquier fuente conmutada.
    Estas interferencias pueden afectar a las líneas de señal.

![ferrite bead](../../img/snubber1.png)
![ferrite bead](../../img/snubber2.png)

!!! success "Solución"
    Se recomienda instalar un **RC snubber (snubber)** en paralelo con el ventilador. O usar un filtro de ferrita

---

## 4. Puerto USB 3.0: problemas durante el uso

!!! warning "Síntomas"
    - Desconexiones periódicas durante el funcionamiento  
    - El dispositivo "desaparece" del sistema sin una causa visible  
    - El problema desaparece al cambiar a otro puerto  

!!! info "Causa"
    Este es un problema común de los dispositivos USB que funcionan en modo Full Speed (USB 2.0) cuando se conectan a puertos USB 3.0. En los ordenadores modernos, los puertos USB 3.0 usan repetidores eUSB2 que no son totalmente compatibles con la especificación USB 2.0; esto provoca fallos de sincronización y errores de enumeración del dispositivo. El problema ha sido confirmado oficialmente por STMicroelectronics: [FAQ en el sitio de ST](https://community.st.com/t5/stm32-mcus/faq-possible-communication-failure-between-stlink-v3-and-some/ta-p/736578).

!!! success "Solución"
    - Conecte iHeater **solo a puertos USB 2.0** (normalmente conectores negros)  
    - Si todos los puertos son USB 3.0, use un **hub USB activo con puertos USB 2.0**

---

## 5. Puerto USB 3.0: problemas durante el flasheo

!!! warning "Síntomas"
    - El controlador no se detecta en modo DFU  
    - El flasheo termina con un error o se queda bloqueado  
    - `dfu-util` no ve el dispositivo o interrumpe la escritura  

!!! info "Causa"
    Es el mismo problema de compatibilidad USB 3.0 / xHCI. Es especialmente relevante al flashear mediante puertos USB Type-C en portátiles modernos, ya que suelen usar repetidores eUSB2 problemáticos.

!!! success "Solución"
    - Al flashear, conecte el controlador **solo a un puerto USB 2.0**  
    - Prefiera los puertos USB Type-A del panel trasero del PC  
    - Si el problema persiste, use un **hub USB activo con puertos USB 2.0**

    
