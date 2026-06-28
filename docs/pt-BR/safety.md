# Segurança

!!! danger "Trabalho com tensão da rede elétrica"
    O dispositivo contém componentes energizados com 110-230 V. Antes de qualquer trabalho elétrico, desligue a alimentação. Certifique-se de que todas as conexões estejam devidamente isoladas antes da primeira energização.

O firmware do controlador — Klipper ou Standalone — implementa proteção por software:

- controle de temperatura usando termistores;
- verificação da presença de conexão dos sensores de temperatura;
- proteção contra temperaturas fora dos limites seguros;
- uso de temporizadores em caso de travamento do sistema;
- desligamento automático em caso de erros dos sensores ou do controlador.

Além disso, há proteção por hardware:

Um Thermal Protector KSD9700 (135 °C) está instalado; em caso de superaquecimento, ele desconecta fisicamente a alimentação do elemento de aquecimento. Quando a temperatura cai abaixo do valor limite, o dispositivo fecha automaticamente o circuito, restaurando a alimentação.

O controlador é equipado com um fusível de 2 A, que protege o dispositivo; em uma situação de emergência, ele queima, desenergizando completamente o sistema.

É utilizado um elemento de aquecimento PTC com isolamento elétrico completo. Ao contrário da maioria das soluções de aquecimento, o corpo do aquecedor PTC não fica energizado, o que elimina o risco de choque elétrico durante a instalação e a manutenção da câmara da impressora 3D.

Esse sistema de proteção em vários níveis torna o iHeater uma solução segura para aquecimento ativo de câmaras de impressoras 3D, inclusive durante operação contínua prolongada.

!!! warning "Instalação do termistor"
    Certifique-se de que as partes expostas dos fios na base do termistor não entrem em contato com a carcaça metálica do aquecedor. Se necessário, isole essas áreas com fita Kapton ou coloque-as em um tubo de teflon / termo-retrátil.

    A temperatura do aquecedor pode atingir 140 °C.

!!! danger "KSD9700 — não é a proteção final"
    O KSD9700 (Thermal Protector) é um dispositivo autorrearmável: em caso de superaquecimento, ele abre o circuito, mas assim que a temperatura cai abaixo do limite, ele o fecha automaticamente de novo. Em caso de falha do aquecedor, o dispositivo irá superaquecer e esfriar ciclicamente sem qualquer intervenção. Isso não é um desligamento de emergência — é um ciclo infinito de superaquecimento.

    Para operação permanente, substitua o KSD9700 por um Thermal Fuse descartável (por exemplo, **RH130**). Ele abre o circuito permanentemente ao ser acionado — o dispositivo é desenergizado e permanece em um estado seguro até a substituição.

!!! note "Procedimento recomendado"
    Use o KSD9700 durante a etapa de montagem e depuração. Após verificar o funcionamento, substitua-o por um Thermal Fuse.
