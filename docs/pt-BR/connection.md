## Diagrama de conexão

O controlador **iHeater** pode funcionar tanto como parte do sistema **Klipper** (como um MCU adicional) quanto de forma autônoma — sob controle do firmware integrado **standalone**.

### Conexão para operação com Klipper

Para o funcionamento correto como parte do Klipper, é necessário conectar:

* **Cabo USB** ao host principal (Host-MCU) - por ele são realizadas a transmissão de dados e a alimentação de 5V;
* **Alimentação de potência 220V / 110V** - dependendo da versão do dispositivo e do tipo de aquecedor;
* **Termistor do aquecedor** - para controlar a temperatura do elemento de aquecimento;
* **Termistor da câmara** - para controlar a temperatura do ar na câmara da impressora;
* **Porta de trigger** - conexão opcional, usada para controle automático a partir de um sinal externo.

Em estado de operação, o iHeater fica instalado dentro da câmara da impressora 3D.

!!! note annotate "Recomenda-se posicionar o termistor da câmara na altura da cabeça de impressão, se possível - **acima da mesa**."

![Diagrama de conexão](../img/iHeater_pinout.png)

## Configuração GPIO

| Pin    | Alias       | Function                                  |
|--------|-------------|-------------------------------------------|
| PA0    | TH1         | Sensor de temperatura da câmara           |
| PA1    | HEATER      | Controle do aquecedor                     |
| PA2    | FAN         | Controle da ventoinha                     |
| PA3    | TH0         | Sensor de temperatura do aquecedor        |
| PA4    | MODE        | Botão de modo                             |
| PA5    | LED3        | LED 3                                     |
| PA6    | LED2        | LED 2                                     |
| PA7    | LED1        | LED 1                                     |
| PB1    | TH2         | Sensor de temperatura adicional           |

---

### Uso no modo standalone

No modo autônomo, estão disponíveis funções e formas de conexão adicionais:

* **Porta de trigger no modo termistor**
  Ao conectar um termistor à porta de trigger e posicioná-lo próximo ao elemento de aquecimento da mesa, é possível ativar o controle automático:
  - quando a mesa aquece acima de **45°C** - o aquecimento da câmara é ligado;
  - quando a temperatura cai abaixo de **85°C** - o aquecimento é desligado.

* **Alimentação por fonte externa de 5V**
  Caso não seja possível usar a alimentação via USB, a alimentação pode ser fornecida diretamente pelo conector correspondente.

* **Opção experimental**
  Conexão de uma [fonte de alimentação de 5V diretamente na placa](https://sl.aliexpress.ru/p?key=OHtN3Xm).

!!! danger "Não use USB e fonte de alimentação externa simultaneamente"
    A conexão simultânea de duas fontes de alimentação não é permitida. Isso causará conflito entre as fontes de alimentação, erros no funcionamento do dispositivo e poderá danificar o equipamento.

![Conexão da alimentação](../img/IMG_6009.jpg)

---

### Diagrama geral de conexão

![Diagrama de conexão](../img/iHeater_connection.png)
