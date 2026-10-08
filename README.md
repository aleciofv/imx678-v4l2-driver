# Driver de kernel para IMX678

Este guia contém instruções detalhadas para instalar o driver de kernel do IMX678 em um sistema Linux, especificamente no Raspbian.

## Agradecimentos especiais

Agradecimentos especiais ao projeto Raspberry Pi CM4 Carrier with Hi-Res MIPI Display, de Sasha Shturma. O script de instalação do DKMS foi adaptado do projeto disponível em: https://github.com/renetec-io/cm4-panel-jdi-lt070me05000


## Pré-requisitos

Antes de iniciar a instalação, verifique se os seguintes pré-requisitos foram atendidos:

- **Versão do kernel**: é necessário usar a versão 6.1 ou mais recente do kernel Linux. Para verificar a versão, execute `uname -r` no terminal.

- **Ferramentas de desenvolvimento**: ferramentas essenciais como `gcc`, `dkms` e `linux-headers` são necessárias para compilar um módulo do kernel. Se ainda não estiverem instaladas, use o gerenciador de pacotes com o seguinte comando:
  
   ```bash 
   sudo apt install linux-headers dkms git
   ```
   
## Etapas de instalação

### Instalação das ferramentas

Primeiro, instale as ferramentas necessárias (`linux-headers`, `dkms` e `git`), caso ainda não tenha feito isso:

```bash 
sudo apt install linux-headers dkms git
```

### Obtenção do código-fonte

Clone o repositório para sua máquina local e acesse o diretório clonado:

```bash
git clone https://github.com/will127534/imx678-v4l2-driver.git
cd imx678-v4l2-driver/
```

### Compilação e instalação do driver de kernel

Para compilar e instalar o driver de kernel, execute o script de instalação fornecido:

```bash 
./setup.sh
```

### Atualização da configuração de inicialização

Edite o arquivo de configuração de inicialização com o seguinte comando:

```bash
sudo nano /boot/config.txt
```

No editor aberto, localize a linha que contém `camera_auto_detect` e altere o valor para `0`. Em seguida, adicione a linha `dtoverlay=imx678`. O resultado deve ficar assim:

```
camera_auto_detect=0
dtoverlay=imx678
```

Depois de fazer essas alterações, salve o arquivo e saia do editor.

Reinicie o sistema para que as alterações entrem em vigor.

## Opções do dtoverlay

### cam0

Se a câmera estiver conectada à porta cam0, acrescente `,cam0` ao dtoverlay, desta forma:
```
camera_auto_detect=0
dtoverlay=imx678,cam0
```

### always-on

Se quiser manter a câmera sempre ligada (útil para depurar problemas de hardware; essa opção mantém o CAM_GPIO constantemente em nível alto), acrescente `,always-on` ao dtoverlay, desta forma:
```
camera_auto_detect=0
dtoverlay=imx678,always-on
```

### Número de lanes

Para usar o IMX678 com 2 lanes, acrescente `,2lane` ao dtoverlay, desta forma:
```
camera_auto_detect=0
dtoverlay=imx678,2lane
```


### Frequência do link

Para alterar a frequência padrão do link, de 1440 Mbps/lane (720 MHz), defina o valor conforme o exemplo a seguir:
```
camera_auto_detect=0
dtoverlay=imx678,link-frequency=297000000
```
Veja a lista de frequências disponíveis:
| Valor de frequência válido | Mbps/lane | Taxa máxima de quadros em 4K 12 bits + 4 lanes | Taxa máxima de quadros em 4K 12 bits + 2 lanes |
| -------- | -------- | -------- | -------- |
| 297000000|594 Mbps/Lane| 20.8 fps | 10.4 fps|
| 360000000|720 Mbps/Lane| 25.0 fps | 12.5 fps|
| 445500000|891 Mbps/Lane| 30.0 fps | 15.0 fps|
| 594000000|1188 Mbps/Lane| 41.7 fps| 20.8 fps|
| 720000000|1440 Mbps/Lane| 50.0 fps | 25.0 fps|
| 891000000|1782 Mbps/Lane| 60.0 fps | 30.0 fps|
| 1039500000|2079 Mbps/Lane| 75.0 fps | 37.5 fps|

Observe que, por padrão, o RPI5/RP1 tem um limite de processamento de 400 Mpix/s. Sem fazer overclock do RP1 (e, portanto, do Camera Frontend), a taxa ficará limitada a aproximadamente 43,8 FPS em 4K.
No modo ClearHDR, a taxa de quadros será reduzida pela metade; em 1080p com binning 2x2, ela será duplicada.
A frequência de 1188 MHz (2376 Mbps/lane) também está disponível no driver, mas, de acordo com os testes, não é compatível com o RPI4 e causa perda de quadros no RPI5.

### Uso combinado

Por fim, todas as opções podem ser usadas simultaneamente. Nesse caso, o dtoverlay ficará assim:
```
camera_auto_detect=0
dtoverlay=imx678,always-on,cam0,link-frequency=297000000
```
