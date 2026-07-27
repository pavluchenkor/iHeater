# Sobre o projeto iHeater

iHeater é um aquecedor compacto para criar uma câmara térmica ativa em uma impressora 3D. É especialmente útil em modelos com eletrônica fechada ou proprietária — Creality, Bambu Lab, FlashForge — onde não há conectores livres para ligar um aquecedor, ventilador e termistor.

Conecta-se por USB e funciona independentemente das limitações da placa principal. Dependendo do firmware — com integração completa ao Klipper ou de forma autônoma.

Em conjunto com o aquecimento da mesa, o iHeater garante aquecimento uniforme da câmara — um fator essencial ao imprimir ABS, PA, PC e outros plásticos de engenharia. O dispositivo controla dinamicamente o aquecimento de acordo com a temperatura do ar, criando condições estáveis dentro da câmara sem superaquecimento nem oscilações.

Duas versões estão disponíveis:

- 100 W — para impressoras pequenas (arquivada)
- 200 W — para impressoras maiores

![iHeater](../img/iHeater_promo.png)

[Você pode fazer um cálculo preliminar usando esta calculadora](https://docs.google.com/spreadsheets/d/1u6XrWLFZGOUnRlFPjjGsJB_GLuFFIs3fFWLCp2-K8gc/edit?usp=sharing)

## Opções de uso

### Sob controle do Klipper

A placa funciona como um MCU separado no Klipper, controlando de forma totalmente autônoma o aquecimento da câmara e o ventilador. A alimentação de 220 V não sobrecarrega a fonte de alimentação da impressora — as fontes originais frequentemente operam no limite.

![PCB](../img/iHeater_200_PCB.png)

O custo da placa é comparável ou inferior ao de montar uma solução semelhante por conta própria com base em um microcontrolador, relé de estado sólido e os componentes necessários. Para entusiastas, continua existindo a possibilidade de montar um análogo por conta própria.

### Com firmware iHeater

A placa iHeater é autossuficiente e contém todos os periféricos necessários para uso como dispositivo independente. A temperatura-alvo é definida por pressionamentos sequenciais do botão MODE e exibida por três LEDs.

## Licença

O projeto é distribuído sob a licença MIT. Detalhes — no arquivo [LICENSE](license.md).

!!! danger "Trabalho com elementos de aquecimento"
    O uso de elementos de aquecimento e o controle de temperatura estão associados ao risco de incêndio e danos ao equipamento. Siga as medidas de precaução. Saiba mais — na seção [Segurança](safety.md).
