# Segurança

!!! danger "Trabalho com tensão da rede elétrica"
    O dispositivo contém componentes sob tensão de 110–230 V. Antes de qualquer trabalho elétrico, desligue a alimentação. Certifique-se de que todas as conexões estejam devidamente isoladas antes da primeira ativação.

O firmware do controlador — Klipper ou Standalone — implementa proteção por software:

- controle de temperatura por meio de termistores;
- verificação da presença de conexão dos sensores de temperatura;
- proteção contra temperaturas fora dos valores seguros;
- uso de temporizadores em caso de travamento do sistema;
- desligamento automático em caso de erros dos sensores ou do controlador.

Além disso, foi implementada proteção por hardware:

Está instalado um Thermal Protector KSD9700 (135 °C), que, em caso de superaquecimento, desliga fisicamente a alimentação do elemento de aquecimento. Quando a temperatura cai abaixo do valor limite, o dispositivo fecha automaticamente o circuito, restaurando a alimentação.

O controlador é equipado com um fusível de 2 A, que protege o dispositivo; em uma situação de emergência, ele queima, desenergizando completamente o sistema.

É utilizado um elemento de aquecimento PTC com isolamento elétrico completo. Ao contrário da maioria das soluções de aquecimento, o corpo do aquecedor PTC não fica sob tensão, o que elimina o risco de choque elétrico durante a instalação e a manutenção da câmara da impressora 3D.

Esse sistema de proteção em vários níveis torna o iHeater uma solução segura para o aquecimento ativo de câmaras de impressoras 3D, inclusive durante operação contínua prolongada.

!!! warning "Instalação do termistor"
    Certifique-se de que as partes expostas dos fios na base do termistor não entrem em contato com o corpo metálico do aquecedor. Se necessário, isole essas áreas com fita Kapton ou coloque-as em um tubo de teflon / termo-retrátil.

    A temperatura do aquecedor pode atingir 140 °C.

!!! danger "KSD9700 — não é a proteção final"
    KSD9700 (Thermal Protector) é um dispositivo autorrearmável: em caso de superaquecimento, ele abre o circuito, mas assim que a temperatura cai abaixo do limite, fecha-o automaticamente novamente. Em caso de falha do aquecedor, o dispositivo irá superaquecer e esfriar ciclicamente sem qualquer intervenção. Isso não é um desligamento de emergência — é um ciclo infinito de superaquecimento.

    Para operação permanente, substitua o KSD9700 por um Thermal Fuse de uso único (por exemplo, **RH130**). Ele interrompe o circuito permanentemente ao atuar — o dispositivo fica desenergizado e permanece em um estado seguro até a substituição.

!!! note "Ordem recomendada"
    Use o KSD9700 durante a etapa de montagem e depuração. Após verificar o funcionamento, substitua-o por um Thermal Fuse.
