# Configuração do Klipper

Esta página descreve a instalação dos arquivos de configuração do iHeater e a configuração do funcionamento com o Klipper.

## Requisitos

### Hardware
  - Placa de controle iHeater
  - Termistores NTC 100K 3950 (2 unid.)
  - Elemento de aquecimento PTC 220V 100W, para a câmara
  - Ventilador 7530 220V, para circulação de ar na câmara
  - Thermal Protector KSD9700 ou equivalente (220 V, 5 A, 130 °C)

### Software
  - Klipper (versão mais recente)
  - Host configurado e em funcionamento com Klipper

## Configuração do Klipper


Copie os arquivos de configuração iHeater.cfg para a pasta com o arquivo printer.cfg (pode ser /klipper_config) e conecte-o em printer.cfg usando a diretiva [include]


```
cd ~/klipper_config
```

```
wget https://raw.githubusercontent.com/pavluchenkor/iHeater/refs/heads/main/iHeater.cfg
```

Abra printer.cfg e adicione

    [include iHeater.cfg]

## Conexão do MCU iHeater

Altere o arquivo iHeater.cfg, informe o ID obtido

```
    [mcu iHeater]
    serial: /dev/serial/by-id/usb-Klipper_stm32f042x6_ХХХХХХХХХХХХХХХХХХХХХХХ-ХХХХ

```

## Preparação para uso

O arquivo de configuração contém a seção:

```ini
[gcode_macro CHAMBER_VARS]
variable_chamber_target: 0          # Целевая температура камеры, °C
variable_start_offset: 10           # Температура камеры, достаточная для начала печати, °C
variable_delta_temp: 10             # Разница между температурой камеры и нагревателя, °C
variable_min_heater_temp: 50        # Минимальная температура нагревателя (для охлаждения), °C
variable_max_heater_temp: 100       # Максимальная температура нагревателя, °C
variable_control_interval: 1.0      # Интервал вызова функции управления, секунды
variable_air_min_delta: 0.5         # Минимальная разница между целевой и текущей температурой камеры (нагреватель = целевая + delta_temp), °C
variable_air_max_delta: 5.0         # Максимальная разница между целевой и текущей температурой камеры (нагреватель = max_heater_temp), °C
gcode:
```

**A temperatura máxima permitida do aquecedor depende do material do gabinete.**

Para verificar:

!. Ligue o aquecimento da mesa a 90-100°C
1. Defina a temperatura do aquecedor para 100°C pela interface Fluidd ou Mainsail.
2. Certifique-se de que o iHeater esteja dentro do volume fechado da impressora.
3. Após atingir a temperatura definida, verifique as áreas onde o aquecedor entra em contato com elementos plásticos do gabinete. O plástico não deve amolecer.
4. Aumente a temperatura em 5-10°C e repita a verificação.
5. Repita até atingir a temperatura máxima permitida do aquecedor sem risco de deformação do gabinete.

Essa abordagem permite determinar um máximo de temperatura seguro e alcançar a melhor eficiência de funcionamento do iHeater.


## Uso

### Comandos de controle de aquecimento da câmara
- Definir a temperatura da câmara:
 

        M141 S60  ; Устанавливает температуру камеры на 60°C

- Aguardar até atingir a temperatura:

        M191 S60  ; Ждет, пока температура камеры достигнет 60°C

- Parar o aquecimento da câmara:

        iHEATER_OFF   ; Отключает нагрев камеры

- No final do G-code do slicer, adicione `iHEATER_OFF` para desligar corretamente o aquecimento da câmara.

### G-code inicial

Slicers modernos oferecem suporte à ativação automática da termocâmara ativa ao gerar o G-code de impressão. Para isso, nas propriedades do filamento é necessário especificar a temperatura da câmara. Caso o slicer não tenha essa funcionalidade, é necessário adicionar ao G-code inicial o comando para ativar o aquecimento da termocâmara ativa.

Procedimento:

- Definir a temperatura-alvo da câmara
- Ligar o aquecimento da mesa para aquecer a câmara de forma eficiente e rápida 
- Continuar o G-code inicial padrão de impressão

Exemplo de G-code inicial
```
; --- Начало стартового G-code ---

; ****** Старт iHeater ******
M141 S60       ; Установить температуру камеры на 60°C
; ****** Конец блока iHeater ******

; --- Остальной стартовый g-code ---
; Включение нагрева стола
...
```
!!! warning "Para a conclusão correta do funcionamento do macro de controle do iHeater, é necessário adicionar o comando iHEATER_OFF ao G-code final da impressora"

```
; --- Начало завершающего g-code ---

; ****** Старт блока iHeater ******
iHEATER_OFF
; ****** Конец блока iHeater ******

; --- Остальной завершающий g-code ---
...
```
## Desativação

Para desativar o iHeater no arquivo printer.cfg, é necessário comentar a linha [include iHeater.cfg]
```
# [include iHeater.cfg]
```

E remover do G-code inicial e final as linhas correspondentes

## Observações
- Segurança:

    - Certifique-se de que todas as conexões estejam feitas corretamente e com segurança.
    - Verifique se os valores min_temp e max_temp correspondem às especificações do equipamento.

- Verificação do equipamento:
    - Antes do uso, teste o funcionamento do aquecedor e do ventilador.
    - Monitore a temperatura durante as primeiras inicializações.
- Configuração de PID:
    - Se necessário, execute a calibração de PID para controle preciso da temperatura.
