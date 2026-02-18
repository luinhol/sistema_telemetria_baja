# Projeto de Telemetria com Ebyte E220 e ESP32

Este repositório contém um projeto de telemetria utilizando comunicação sem fio baseada em módulos **Ebyte E220-900T22D** e microcontroladores **ESP32**. O sistema inclui transmissão de dados via ESP32, recepção via outro ESP32 conectado ao computador e leitura dos dados usando Python.

---

## Objetivo

O projeto tem como objetivo implementar e testar um sistema de telemetria ponto a ponto, coletando e monitorando dados de sensores de um veículo baja SAE em tempo real. Ele permite:

- Transmissão de dados de sensores via LoRa.
- Recepção de dados em um computador para análise.

---

## Dispositivos Utilizados

- **2x Ebyte E220** – módulos LoRa para comunicação sem fio.
- **ESP32** – microcontrolador responsável por transmitir os dados.
- **Computador** – para leitura e análise dos dados recebidos.

---

## Estrutura do Repositório

```
.
├── README.md                 # Este arquivo
├── scripts/                  # Scripts e programas principais
│   ├── lora_transmissor_one_way_48_128/
│   ├── lora_transmissor_one_way_48_128.ino
│   └── leitura_serial.py
├── essential/                # Fotos, esquemas e documentação adicional
│   ├── adaptador_EBYTE/
│   └── programa_configurar_lora/
├── analise/                  # Programa base de análise de dados coletados
├── resultados/               # Dados coletados durante testes
└── relatorio/                # Relatório parcial ou final do projeto
```

---

## Como Utilizar

### 1. Configuração dos Módulos Ebyte E220
Para preparar os módulos E220 antes do uso no projeto de telemetria, siga os passos abaixo:

#### 1. Preparação do módulo
- Desconecte os jumpers dos pinos **M0** e **M1** do adaptador para colocar o módulo em **modo de configuração**.
- Selecione **3.3 V** no jumper de alimentação do adaptador.

#### 2. Conectar antena
- Fixe uma antena no módulo E220. Isso é importante para evitar danos ao módulo durante a configuração.

#### 3. Conectar ao computador
- Insira o módulo E220 no adaptador, com a antena voltada para fora.
- Conecte o adaptador a uma **porta USB** do computador.

#### 4. Abrir o software de configuração
- Abra o aplicativo de configuração do E220. (essentials/programa_configurar_lora/e220 V1.3.exe)
- Selecione a **porta COM** correspondente à porta USB utilizada.
- Clique no botão **Open** para conectar ao módulo.
- Clique em **Get** para ler os parâmetros atuais do módulo.
- Alternativamente, utilize **Param Reset** para restaurar os parâmetros padrão. (ATENÇÂO: Apenas para restaurar ao padrão de fábrica)

#### 5. Configurar os parâmetros
- Modifique o **canal** para **65** (resultando na frequência de 915.125 MHz, compatível com a antena utilizada).
- Ative a função **Packet RSSI**, para que cada pacote enviado inclua um byte com o valor do RSSI.
- Outros parâmetros podem ser ajustados conforme necessidade para otimizar o desempenho.

#### 6. Salvar e finalizar
- Clique em **Set Param** para gravar os novos parâmetros no módulo.
- Clique em **Close** e desconecte o módulo com segurança.
- Reconecte os jumpers dos pinos **M0** e **M1** do adaptador para colocar o módulo em **modo de transmissão**.
- O módulo agora está pronto para uso no projeto de telemetria.

### 2. Código Transmissor (ESP32)
1. Abra `lora_transmissor_one_way_48_128.ino` na Arduino IDE ou PlatformIO.
2. Configure a porta e o microcontrolador correto (ESP-32S). 
3. Faça upload para o ESP32 transmissor.
4. O ESP32 começará a enviar os dados via LoRa.

### 3. Código Receptor e Leitura Serial
1. Conecte o módulo receptor ao computador via USB.
2. Execute `leitura_serial.py`:
   python leitura_serial.py

## Alterações e Ajustes Necessários

O projeto ainda requer algumas modificações e ajustes para funcionar em comunicação com o veículo Baja SAE:

### São necessários as seguintes modificações:

#### No código do STM32
1. Conexão serial entre o STM32 e o ESP32.
2. Envio de uma mensagem serial do STM32 para o ESP32 com os dados necessários a cada 0.2s.

#### No lora_transmissor_one_way_48_128.ino
1. Montagem de um pacote de 128 bytes com 5 amostras de dados. (Substituindo o gps e trocando o padding pelos dados).

#### No leitura_serial.py
1. Modificar o struct.unpack para receber todo o pacote com todos os dados necessários.
2. Dividir os dados em diferentes variáveis e escrever linha a linha no arquivo.

## Sugestões

Ao fazer a integração entre STM32 e ESP32, é possível transferir tarefas realizadas pelo STM32 para o ESP32, tais como: Adquirir dados de baixa amostragem (GPS), mostrar dados no display, entre outros.

A integração entre STM32 e ESP32 pode ser feita por serial, i2c ou spi. Cabe a equipe decidir a melhor maneira de fazer essa conexão, minha sugestão é começar com serial, mas fazer testes com outras interfaces.

Uma sugestão é o posicionamento da antena, o ideal é que ela fique no topo do carro, se permitido pelo regulamento. E que não seja usado longos cabos coaxiais, eles degradam rapidamente o sinal, porém pode ser importante ter um cabo para garantir flexibilidade em caso de capotamento.

É importante testar novos modelos de antena, o manual do E220 indica as antenas: TX915-JKS-20 e TX915-XPL-100, minha recomendação é comprar as duas antenas, testar e avaliar a que melhor atende.

Também deixo a sugestão de posicionar apenas o módulo com a antena na parte superior do carro, mantendo o ESP32 e o STM32 juntos no painel do veículo, dessa maneira, deve ter um cabo que faz a interface serial entre o ESP32 e o módulo LoRa.

Ressalto que o sistema é sensível à agua, diante disso, recomendo estudo de soluções com boa vedação, mesmo que em impressão 3D.

Por último, recomendo a criação de um pipeline para tratamento e visualização dos dados, em minha pesquisa o que achei mais interessante seria a combinação de MQTT Broker + Telegraf + InfluxDB + InfluxDBCloud + Grafana, tudo isso encapsulado em um Docker. Estou deixando um diagrama de sugestão no diretório essencials.

## Recados finais

Esse é um primeiro de telemetria para o projeto, diante disso, recomendo que utilizem como base, porém não se prendam totalmente às escolhas e soluções aqui apresentadas, testem o sistema proposto, procurem melhorias, testem novamente, testem novas configurações, novos módulos...

Obrigado por todo esse tempo que passei na equipe Vitória Baja.