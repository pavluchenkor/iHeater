# Aquecimento da câmara térmica em uma impressora 3D

## 1. Verificação dos ventiladores e aberturas de ventilação

* Em modelos novos de impressoras, frequentemente são instalados ventiladores para expulsar o ar quente da câmara. Antes de iniciar o uso, certifique-se de que eles não liguem automaticamente quando a temperatura da câmara for excedida..
* Verifique as frestas no gabinete da impressora e ao redor da porta
* Verifique as aberturas de ventilação do gabinete. Se a câmara tiver contato com o compartimento da eletrônica e houver aberturas não fechadas ali, elas precisam ser vedadas:

    * com fita adesiva comum;
    * com fita adesiva de alumínio (reflete melhor o calor);
    * ou com isolamento térmico (melhor opção).
* Isso é necessário para que a eletrônica de controle não superaqueça.


## 2. Fonte de calor: mesa da impressora

* A mesa aquecida é a principal fonte de calor para a câmara.
* O iHeater, por si só, normalmente não elevará a temperatura ao valor necessário sem a mesa.
* Se for necessário aquecer o volume da impressora sem a mesa ligada, use aquecedores de fabricação industrial com potência de 600 W a 1 kW para compensar a ausência de calor da mesa (**com verificação de segurança elétrica e da capacidade de carga dos circuitos de alimentação**).

## 3. Uso de coolers auxiliares

* Em modelos modernos, frequentemente há ventiladores adicionais ao longo das paredes laterais, destinados ao resfriamento adicional do modelo, ou filtros de carvão dentro da câmara.
* Na etapa de aquecimento da câmara, eles podem ser ligados para misturar o ar - isso acelera e uniformiza o aquecimento.
* O ideal é prever uma lógica de macro: no início do aquecimento, os coolers são ligados; após atingir a temperatura-alvo ou as camadas iniciais da impressão, são desligados.

## 4. Quando iniciar a impressão

* Exemplo: temperatura-alvo da câmara - 60°C.
* A impressão pode ser iniciada em 50-55°C, porque antes do início há operações preparatórias: criação do mapa da mesa, aquecimento e limpeza do bico, deposição das primeiras camadas.
* Esses processos levam vários minutos; durante esse tempo, a câmara consegue se aproximar do valor-alvo.
* Depois da impressão dos primeiros 2-3 mm do modelo, a câmara geralmente atinge a temperatura necessária e se estabiliza; esse parâmetro é individual para cada impressora

## 5. Instalação do termistor

* Posicione o sensor de temperatura (termistor) aproximadamente na altura da cabeça de impressão.
* Ele não deve tocar nas peças do gabinete da impressora; caso contrário, lerá a temperatura delas, e não a temperatura do ar.

## 6. Segurança e recomendações adicionais

* Posicione o iHeater de modo a garantir um fluxo uniforme e evitar superaquecimentos locais.
* Aplique isolamento térmico no gabinete para reduzir perdas de calor.
* As linhas de alimentação dos aquecedores devem suportar a carga de corrente (cabo, conectores, fusível).
* Os regimes de temperatura devem corresponder aos materiais do gabinete da impressora e às condições de operação.

---

## 7. Checklist curto antes da inicialização

* [ ] Os ventiladores estão funcionando; as aberturas de ventilação estão limpas.
* [ ] O compartimento da eletrônica está isolado do volume quente da câmara.
* [ ] A fonte de calor adicional está ligada.
* [ ] Os coolers para mistura do ar são ativados na etapa de pré-aquecimento.
* [ ] O sensor de temperatura está instalado na altura da cabeça de impressão e não toca no gabinete.
* [ ] Os limites de temperatura e os desligamentos de emergência estão configurados.
