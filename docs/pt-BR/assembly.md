# Montagem

Leia a documentação, baixe e imprima as peças necessárias. Certifique-se de ter todos os componentes e ferramentas antes de iniciar a montagem.

!!! danger "Trabalho com tensão da rede elétrica"
    Todos os trabalhos de conexão à rede de 110–230 V devem ser realizados com o dispositivo desenergizado. Mais detalhes na seção [Segurança](safety.md).

## Antes da montagem

Recomenda-se primeiro montar todo o sistema **na bancada**, sem instalá-lo na carcaça, e realizar os testes:

- Conectar **todos** os componentes.
- Verificar o funcionamento do aquecedor, do ventilador e dos sensores de temperatura.
- Conectar o sistema ao **Klipper** ou gravar o firmware Standalone e confirmar que tudo funciona corretamente.

Video guide: [YouTube](https://youtu.be/1QMtVY0Vx-8?si=Ol1u4Ux9wALDcfe2)

## Montagem passo a passo

### Instalação da placa

![Montagem do iHeater](../img/iHeater_5484.jpg)

### Instalação do termistor e do Thermal Protector

!!! warning "Instalação do termistor"
    Certifique-se de que as partes desencapadas dos fios na base do termistor não entrem em contato com a carcaça metálica do aquecedor. Se necessário, isole essas áreas com fita Kapton ou coloque-as em um tubo de teflon / tubo termo-retrátil.

    A temperatura do aquecedor pode chegar a 140 °C.

!!! warning "Instalação do Thermal Protector"
    É possível instalar um KSD9700 (Thermal Protector, autorrearmável) ou um Thermal Fuse descartável.

    O KSD9700 abre o circuito em caso de superaquecimento e o fecha automaticamente após o resfriamento. O Thermal Fuse (por exemplo, **RH130**) interrompe o circuito permanentemente ao ser acionado — uma proteção mais confiável em caso de falha.

    Use o KSD9700 durante a etapa de depuração e depois substitua-o por um Thermal Fuse para operação permanente.

![Montagem do iHeater](../img/iHeater_5489.jpg)
![Montagem do iHeater](../img/thermistor.jpg)

### Instalação do aquecedor

!!! warning "Instalação do termistor"
    Instale o termistor na borda do aquecedor, aproximadamente na metade da altura das aletas do dissipador.

    As partes desencapadas dos fios na base do termistor não devem tocar a carcaça metálica do aquecedor. Se necessário, isole essas áreas com fita Kapton ou coloque-as em um tubo de teflon / tubo termo-retrátil.

    A temperatura do aquecedor pode chegar a 140 °C.

![Montagem do iHeater](../img/iHeater_5491.jpg)

### Fiação

![Montagem do iHeater](../img/iHeater_5494.jpg)

### Instalação dos terminais tubulares

![Montagem do iHeater](../img/iHeater_5496.jpg)

### Conexões

![Montagem do iHeater](../img/iHeater_5498.jpg)

### Montagem final

![Montagem do iHeater](../img/iHeater_5500.jpg)

### Produto montado

![Montagem do iHeater](../img/iHeater_5506.jpg)
