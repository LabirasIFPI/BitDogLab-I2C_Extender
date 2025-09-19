# Extensor I2C

## **Fundamentos da Comunicação I2C e Expansão de Barramento**

O pilar deste projeto é a utilização do protocolo I2C para gerenciar múltiplos dispositivos. Compreender seu funcionamento e como expandir suas conexões físicas é essencial.

### 1.1. O Protocolo I2C (Inter-Integrated Circuit)

O I2C é um barramento de comunicação serial síncrono, projetado para a comunicação de curta distância entre circuitos integrados. Sua popularidade em sistemas embarcados deve-se à sua simplicidade e eficiência, exigindo apenas duas linhas de sinal.

- **Arquitetura Mestre-Escravo:** A comunicação opera com um dispositivo Mestre (o Raspberry Pi Pico), que controla o barramento, e um ou mais dispositivos Escravos (sensores, displays), que respondem às solicitações do mestre.
- **Linhas de Sinal:**
  - **SCL (Serial Clock Line):** Linha de clock que sincroniza a transferência de dados, gerada exclusivamente pelo Mestre.
  - **SDA (Serial Data Line):** Linha bidirecional por onde os dados são efetivamente transferidos.
- **O Mecanismo de Endereçamento:** A característica que permite a coexistência de múltiplos escravos em um mesmo par de fios é o **endereçamento**. Cada dispositivo escravo possui um endereço de 7 bits único (fixado pelo fabricante ou configurável por hardware). Toda comunicação iniciada pelo Mestre começa com a transmissão do endereço do escravo alvo. Apenas o escravo que reconhece seu endereço participa da transação de dados subsequente, enquanto os demais permanecem inativos.

### 1.2. O Hub Extensor I2C

O Hub Extensor I2C, é um componente fundamental para a organização física de múltiplos dispositivos em um único barramento.

- **Função Principal:** O hub atua como um **divisor de barramento passivo**. Ele não contém lógica digital, microcontroladores ou "inteligência". Sua função é puramente elétrica: conectar um ponto de entrada a múltiplos pontos de saída em paralelo.
- **Operação Interna:** Internamente, a placa do hub consiste em trilhas que conectam todos os pinos correspondentes. O pino `SCL` da porta de entrada está fisicamente ligado a todos os pinos `SCL` das portas de saída. O mesmo ocorre para `SDA`, `VCC` e `GND`.
- **Analogia Funcional:** Pode-se pensar no hub como uma "régua de tomadas"  para o barramento I2C. Ele transforma uma única porta I2C do microcontrolador em várias portas fisicamente acessíveis.
- **Por que Funciona?** Como explicado no mecanismo de endereçamento, mesmo que o sinal elétrico do Mestre chegue a todos os sensores simultaneamente através do hub, apenas o sensor cujo endereço foi chamado responderá. Portanto, o hub não gera conflitos, apenas simplifica a fiação que, de outra forma, teria que ser feita manualmente (soldando fios em comum). Isso significa que **qualquer sensor pode ser conectado a qualquer porta do hub**, pois todas são eletricamente idênticas.

## 2. Visão Geral do Projeto

Compreendidos os fundamentos, este documento detalha a arquitetura de um sistema embarcado que utiliza os conceitos acima. O projeto integra um acelerômetro (MPU-6050) e um sensor de luminosidade (BH1750), conectados a um Hub I2C, com um servo motor (SG90) e um display OLED (SSD1306), no qual, todos os sensores são conectados e configurados no mesmo barramento i2C, no caso o I2c0 e somente o display está ligado ao I2c1 por padrão da BitDogLab . O sistema lê os dados dos sensores, controla o servo com base na inclinação do acelerômetro e exibe todas as informações em tempo real no display.

## 3. Arquitetura de Hardware do Projeto

A implementação física utiliza as duas interfaces I2C do Raspberry Pi Pico para segregar os periféricos de forma lógica e eficiente.

### 3.1. Barramento `i2c0` (Sensores via Hub Extensor)

- **Pinos do Pico:** GPIO 8 (SDA) e GPIO 9 (SCL).
- **Conexão:** A saída `i2c0` do Pico é conectada à porta de entrada do Hub Extensor I2C.
- **Periféricos Conectados ao Hub:**
  - **MPU-6050 (Acelerômetro/Giroscópio):** Endereço I2C `0x68`.
  - **BH1750 (Sensor de Luminosidade):** Endereço I2C `0x23`.

### 3.2. Barramento `i2c1` (Display)

- **Pinos do Pico:** GPIO 14 (SDA) e GPIO 15 (SCL).
- **Dispositivo:**
  - **SSD1306 (Display OLED):** Endereço I2C `0x3C`.

### 3.3. Controle do Atuador (PWM)

- **Pino do Pico:** GPIO 2.
- **Dispositivo:**
  - **Servo Motor SG90:** Controlado via sinal PWM.

## 4. Arquitetura de Software

O firmware é modular, com drivers dedicados para cada periférico, um módulo de inicialização centralizado.

- **Drivers de Periféricos:** Módulos (`.c`/`.h`) para MPU-6050, BH1750, SSD1306 e Servo encapsulam a comunicação de baixo nível.
- **Módulo de Inicialização (`init.c`):** A função `initializeSystem()` configura ambos os barramentos I2C, o PWM e chama as rotinas de inicialização de cada driver.
- **Aplicação Principal (`main.c`):** Contém o loop principal que sequencialmente lê os sensores, processa os dados, comanda o servo e atualiza o display.
