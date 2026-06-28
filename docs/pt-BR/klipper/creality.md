## Firmware iHeater para impressoras Creality via Creality Helper Script

Para que o firmware e a integração do iHeater sejam concluídos com sucesso, siga as instruções passo a passo:

### 1. Instale o Creality Helper Script

Acesse a página de documentação do projeto Creality Helper Script e siga as instruções de instalação do script.

**Recursos:**

* Guia em vídeo: [YouTube](https://youtu.be/k9kPcDfBgmo?t=254)
* Instrução em texto: [guilouz.github.io](https://guilouz.github.io/Creality-Helper-Script-Wiki/firmwares/install-and-update-rooted-firmware-k1/)


### 2. Obtenha acesso root à impressora e acesso ao sistema de arquivos

O script abrirá o acesso ao Mainsail, bem como aos arquivos de configuração do firmware. Após a instalação bem-sucedida, certifique-se de que você consegue acessar a interface da impressora pelo navegador e obter acesso aos arquivos de configuração.

### 3. Remova o `fan-control.cfg` antigo

Nas impressoras Creality com Helper Script, por padrão já pode existir um arquivo `fan-control.cfg` com os macros `M141` e `M191`. Ele entra em conflito com macros semelhantes na configuração do iHeater.

Renomeie o arquivo:

```
/usr/data/printer_data/config/fan-control.cfg
```
para fan-control.cfg.bak

### 4. Copie o novo `fan-control.cfg`

Substitua-o pela versão [fan-control.cfg](../../../printers/creality/config/fans-control.cfg), compatível com os macros e a lógica de controle da temperatura da câmara.

Coloque o novo arquivo na mesma pasta:

```
/usr/data/printer_data/config/fan-control.cfg
```


### 5. Adicione a configuração do iHeater

Copie o arquivo `iheater.cfg` para o mesmo diretório:

```
/usr/data/printer_data/config/iheater.cfg
```

Em seguida, abra `printer.cfg` e adicione a seguinte linha ao final do arquivo:

```ini
[include iheater.cfg]
```

---

Depois, siga as instruções de configuração do iHeater - configuração do termistor, aquecedor, modos de operação e macros.

!!! warning "Se não for possível compilar e gravar o firmware na impressora"
    [Consulte a seção WSL](../user-mods/software/wsl2-ubuntu-ff/)
