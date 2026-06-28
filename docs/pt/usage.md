# Aquecimento da câmara térmica em uma impressora 3D

## 1. Verificação dos ventiladores e das aberturas de ventilação

* Em modelos novos de impressoras, frequentemente são instalados ventiladores para expulsar o ar quente da câmara. Antes de iniciar a operação, certifique-se de que eles não ligam automaticamente quando a temperatura da câmara é excedida..
* Verifique as frestas no corpo da impressora e ao redor da porta
* Verifique as aberturas de ventilação do corpo. Se a câmara tiver contato com o compartimento da eletrônica e houver aberturas não fechadas, elas devem ser fechadas:

    * com fita adesiva comum;
    * com fita de alumínio (reflete melhor o calor);
    * ou com isolamento térmico (melhor opção).
* Isso é necessário para que a eletrônica de controle não superaqueça.


## 2. Fonte de calor: mesa da impressora

* A mesa aquecida é a principal fonte de calor para a câmara.
* Por si só, o iHeater normalmente não elevará a temperatura até o valor necessário sem a mesa.
* Se for necessário aquecer o volume da impressora sem a mesa ligada, use aquecedores de fabricação industrial com potência de 600W - 1kW para compensar a ausência de calor da mesa (**com verificação da segurança elétrica e da capacidade de carga dos circuitos de alimentação**).

## 3. Uso de coolers auxiliares

* Em modelos modernos, frequentemente há ventiladores adicionais ao longo das paredes laterais, destinados ao resfriamento adicional do modelo, ou filtros de carvão dentro da câmara.
* Na etapa de aquecimento da câmara, eles podem ser ligados para misturar o ar - isso acelera e uniformiza o aquecimento.
* O ideal é prever uma macro-lógica: no início do aquecimento, os coolers ligam; após atingir a temperatura-alvo ou as camadas iniciais da impressão, desligam.

## 4. Quando iniciar a impressão

* Exemplo: temperatura-alvo da câmara - 60°C.
* A impressão pode ser iniciada a 50-55°C, porque antes do início há operações preparatórias: construção do mapa da mesa, aquecimento e limpeza do bico, deposição das primeiras camadas.
* Esses processos levam alguns minutos; nesse tempo, a câmara consegue se aproximar do valor-alvo.
* Após a impressão dos primeiros 2-3mm do modelo, a câmara normalmente atinge a temperatura necessária e se estabiliza; esse parâmetro é individual para cada impressora

## 5. Instalação do termistor

* Posicione o sensor de temperatura (termistor) aproximadamente no nível da cabeça de impressão.
* Ele não deve tocar nas peças do corpo da impressora; caso contrário, lerá a temperatura delas, não a temperatura do ar.

## 6. Segurança e recomendações adicionais

* Posicione o iHeater de forma a garantir um fluxo uniforme e evitar superaquecimentos locais.
* Use isolamento térmico no corpo para reduzir as perdas de calor.
* As linhas de alimentação dos aquecedores devem suportar a carga de corrente (cabo, conectores, fusível).
* Os regimes de temperatura devem corresponder aos materiais do corpo da impressora e às condições de operação.

---

## 7. Checklist curto antes do início

* [ ] Os ventiladores estão em boas condições; as aberturas de ventilação estão limpas.
* [ ] O compartimento da eletrônica está isolado do volume quente da câmara.
* [ ] A fonte de calor adicional está ligada.
* [ ] Os coolers para mistura do ar são ativados na etapa de pré-aquecimento.
* [ ] O sensor de temperatura está instalado no nível da cabeça de impressão e não toca no corpo.
* [ ] Os limites de temperatura e desligamentos de emergência estão configurados.
