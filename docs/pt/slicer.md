## Configuração do G-code inicial no slicer para o funcionamento correto do iHeater

Para obter um aquecimento uniforme e uma impressão estável com materiais técnicos, é importante configurar corretamente a ordem dos comandos no G-code inicial. Abaixo estão recomendações para integrar o aquecimento da câmara iHeater ao código inicial do seu slicer (por exemplo, Cura, PrusaSlicer, OrcaSlicer etc.).

### O que precisa ser feito

Antes de ligar o aquecimento da câmara, é necessário:

**Ligar o aquecimento da mesa**

   O aquecimento da mesa ajuda a câmara a aquecer de forma mais uniforme e mais rápida. Essa mesa atua como uma fonte adicional de calor, contribuindo para um aquecimento mais eficiente de toda a câmara.

   ```
   M140 S[first_layer_bed_temperature]
   ```

**Ligar o ventilador de circulação de ar (se usado)**

   Pode ser um ventilador lateral ou qualquer outro destinado à distribuição uniforme do calor na câmara.
   Se na configuração do Klipper ele se chama, por exemplo, `chamber_fan`, então:

   ```
   SET_FAN_SPEED FAN=chamber_fan SPEED=1.0
   ```

Em algumas impressoras há ventiladores de exaustão que mantêm a temperatura mínima possível na câmara. Isso é importante ao imprimir com plásticos como PLA e PETG, mas prejudica a velocidade de aquecimento da câmara. Para esse ventilador, é possível definir um novo parâmetro de temperatura, por exemplo temperatura_alvo_na_câmara + 10

```
SET_TEMPERATURE_FAN_TARGET TEMPERATURE_FAN=chamber_fan TARGET={chamber_temperature + 10}
```

**Ligar o aquecimento da câmara usando um dos macros: `M141` ou `M191`**

### Diferença entre `M141` e `M191`

| Macro | Finalidade                                                                    | Bloqueia a execução do código                                      |
| ----- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `M141` | Define a temperatura alvo da câmara                                          | Não (o aquecimento começa, mas a impressão continua imediatamente) |
| `M191` | Define a temperatura da câmara e espera até que ela alcance o valor definido | Sim (o próximo comando só será executado após o aquecimento)       |

#### Exemplos:

* Se você deseja iniciar imediatamente o aquecimento da câmara e executar todas as operações preliminares em paralelo com o aquecimento, e depois continuar a impressão sem esperar o aquecimento completo da câmara, esta opção é adequada. Ela funciona bem para plásticos do tipo ABS e modelos grandes - enquanto as primeiras camadas são impressas, a câmara terá tempo para aquecer:

  ```
  M141 S60
  ```

* Se estiver imprimindo uma peça pequena ou se for necessária uma temperatura estável na câmara (por exemplo, ao imprimir PA, PC e outros materiais sensíveis), é melhor usar o comando com espera pelo aquecimento:

  ```
  M191 S60
  ```

### Após o aquecimento da câmara

Depois de chamar um dos macros (`M141` ou `M191`), é possível continuar com o G-code inicial normal:

```gcode
M190 S[first_layer_bed_temperature]  Espera pelo aquecimento da mesa
M109 S[first_layer_temperature]  Espera pelo aquecimento do hotend
G28  Homing
G29  (se a calibração automática for usada)
... 
```

### Recomendações

* Se você usa `M191`, é possível definir um pequeno desvio no qual a câmara é considerada suficientemente aquecida (por exemplo, 5°C abaixo da meta) - isso é configurado no macro [gcode_macro CHAMBER_VARS] `variable_start_offset`.
* Certifique-se de que todos os macros usados (`M141`, `M191`) estejam incluídos na configuração do Klipper e configurados corretamente.
* Se o seu slicer oferece suporte a condições, é possível adicionar uma verificação: por exemplo, ligar a câmara somente quando a temperatura de impressão estiver acima de 50°C (para ABS, ASA etc.).

---

### Configuração da temperatura na câmara térmica no slicer

![Slicer_settings](../img/slicer_settings.jpeg)

Em muitos slicers modernos, é possível especificar a temperatura desejada da câmara térmica diretamente no perfil do filamento. Esse valor é conveniente para usar como parâmetro `S` nos macros `M141` ou `M191`.

A captura de tela mostra o campo "Chamber temperature", no qual o valor `60°C` foi definido. Isso é apenas um valor numérico - **nem todos os slicers enviam o comando para aquecer a câmara**. Para que a temperatura seja realmente aplicada, é necessário verificar a geração do G-code pelo slicer e usar comandos no G-code inicial, se necessário:

```gcode
M141 S{chamber_temperature}
```

ou

```gcode
M191 S{chamber_temperature}
```

Certifique-se também de que a variável `chamber_temperature` esteja definida nas configurações do slicer, ou substitua-a manualmente por um número.

Se a opção "Activate temperature control" for ativada, alguns slicers adicionam automaticamente o comando `M191` com o valor de temperatura definido. Isso é conveniente se você quiser que o aquecimento da câmara aconteça com espera antes da impressão.

Recomenda-se usar o controle manual por meio de macros para ter controle total sobre a lógica de aquecimento e a sequência de ações.
