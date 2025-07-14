# Documentação ESP32 IoT System

## Visão Geral

Este projeto implementa um sistema IoT completo usando ESP32 que monitora condições ambientais (luz, temperatura e umidade) e controla um LED automaticamente. Os dados são enviados via MQTT para um broker HiveMQ e exibidos em um display LCD I2C com alternância automática entre telas.

## Funcionalidades Principais

- **Monitoramento de Luz**: Sensor fotossensível que controla LED automaticamente
- **Monitoramento Ambiental**: Sensor DHT11 para temperatura e umidade
- **Conectividade**: WiFi e comunicação MQTT
- **Interface Visual**: Display LCD I2C com rotação automática de telas
- **Controle Remoto**: Recepção de comandos via MQTT

## Bibliotecas Utilizadas

```cpp
#include <WiFi.h>           // Conectividade WiFi
#include <PubSubClient.h>   // Cliente MQTT
#include <LiquidCrystal_I2C.h> // Display LCD I2C
#include <DHT.h>            // Sensor DHT11
#include <ArduinoJson.h>    // Manipulação de JSON
```

## Estrutura de Configuração

### Config (struct)

Estrutura estática que centraliza todas as configurações do sistema:

```cpp
struct Config {
  // Configurações de Rede
  static const char* WIFI_SSID;        // Nome da rede WiFi
  static const char* WIFI_PASSWORD;    // Senha da rede WiFi
  
  // Configurações MQTT
  static const char* MQTT_BROKER;      // Endereço do broker MQTT
  static const int MQTT_PORT = 1883;   // Porta do broker MQTT
  static const char* MQTT_CLIENT_ID;   // ID único do cliente MQTT
  static const char* MQTT_TOPIC_LIGHT; // Tópico para dados de luz
  static const char* MQTT_TOPIC_DHT;   // Tópico para dados DHT
  static const char* MQTT_TOPIC_STATUS; // Tópico para status do sistema
  
  // Configurações de Hardware
  static const int PHOTOSENSOR_PIN = 32;  // Pino do sensor de luz
  static const int LED_PIN = 19;          // Pino do LED
  static const int DHT_PIN = 2;           // Pino do sensor DHT11
  static const int LIGHT_THRESHOLD = 800; // Limiar para ativar LED
  
  // Configurações do Display
  static const int LCD_ADDRESS = 0x27;  // Endereço I2C do LCD
  static const int LCD_COLS = 16;       // Colunas do LCD
  static const int LCD_ROWS = 2;        // Linhas do LCD
  static const int DHT_TYPE = DHT11;    // Tipo do sensor DHT
};
```

**Finalidade**: Centralizar todas as configurações em um local único, facilitando manutenção e modificações.

## Classes do Sistema

### 1. ScreenManager

Gerencia a alternância automática entre telas do LCD.

**Atributos**:
- `currentScreen`: Tela atual (0, 1 ou 2)
- `lastScreenChange`: Timestamp da última mudança
- `screenInterval`: Intervalo entre mudanças (5 segundos)

**Métodos principais**:
- `shouldChangeScreen()`: Verifica se deve alternar tela
- `getCurrentScreen()`: Retorna a tela atual
- `resetTimer()`: Reinicia o timer de alternância

**Finalidade**: Proporcionar interface dinâmica alternando entre informações de luz/LED, temperatura/umidade e status MQTT.

### 2. DHTSensor

Controla o sensor DHT11 para leitura de temperatura e umidade.

**Atributos**:
- `dht`: Instância do sensor DHT
- `temperature`: Temperatura atual
- `humidity`: Umidade atual

**Métodos principais**:
- `initialize()`: Inicializa o sensor
- `readSensor()`: Lê valores do sensor
- `getTemperature()`: Retorna temperatura
- `getHumidity()`: Retorna umidade
- `getTemperatureString()`: Retorna temperatura formatada
- `getHumidityString()`: Retorna umidade formatada

**Finalidade**: Abstrair a comunicação com o sensor DHT11 e fornecer dados formatados.

### 3. WiFiManager

Gerencia a conexão WiFi do ESP32.

**Atributos**:
- `connected`: Estado da conexão WiFi

**Métodos principais**:
- `connect()`: Conecta à rede WiFi
- `isConnected()`: Verifica status da conexão
- `reconnect()`: Reconecta se desconectado
- `getLocalIP()`: Retorna IP local

**Finalidade**: Abstrair o gerenciamento de WiFi com reconexão automática.

### 4. LCDDisplay

Controla o display LCD I2C e suas diferentes telas.

**Atributos**:
- `lcd`: Instância do display LCD
- `animationStep`: Controla animação de conexão

**Métodos principais**:
- `initialize()`: Inicializa o display
- `showMessage()`: Exibe mensagem genérica
- `showConnectingStart()`: Inicia animação de conexão
- `animateConnecting()`: Anima processo de conexão
- `showSensorData()`: Exibe dados do sensor de luz
- `showDHTData()`: Exibe dados de temperatura/umidade
- `showMQTTStatus()`: Exibe status MQTT
- `showMQTTPublishStatus()`: Mostra status de publicação

**Finalidade**: Centralizar todas as operações do display LCD com diferentes layouts de tela.

### 5. LightSensor

Controla o sensor de luz e o LED automático.

**Atributos**:
- `ledState`: Estado atual do LED
- `lastLedState`: Estado anterior do LED
- `lightValue`: Valor atual do sensor de luz

**Métodos principais**:
- `readSensor()`: Lê sensor e controla LED
- `hasStateChanged()`: Verifica mudança de estado
- `getLightValue()`: Retorna valor do sensor
- `getLedState()`: Retorna estado do LED
- `getJsonData()`: Retorna dados em formato JSON

**Finalidade**: Implementar controle automático do LED baseado na luminosidade ambiente.

### 6. MQTTClient

Gerencia comunicação MQTT com o broker.

**Atributos**:
- `client`: Cliente MQTT
- `messagesSent`: Contador de mensagens enviadas
- `lastReconnectAttempt`: Timestamp da última tentativa de reconexão

**Métodos principais**:
- `initialize()`: Configura cliente MQTT
- `connect()`: Conecta ao broker
- `loop()`: Mantém conexão ativa
- `publishLightData()`: Publica dados de luz
- `publishDHTData()`: Publica dados DHT
- `publishStatus()`: Publica status do sistema
- `onMessageReceived()`: Callback para mensagens recebidas

**Finalidade**: Abstrair comunicação MQTT com reconexão automática e publicação de dados.

### 7. IoTSystem

Classe principal que coordena todo o sistema.

**Atributos**:
- `wifiManager`: Gerenciador WiFi
- `display`: Controlador do display
- `sensor`: Sensor de luz
- `dhtSensor`: Sensor DHT
- `screenManager`: Gerenciador de telas
- `mqttClient`: Cliente MQTT
- `lastDHTPublish`: Timestamp da última publicação DHT

**Métodos principais**:
- `initialize()`: Inicializa todo o sistema
- `loop()`: Loop principal do sistema
- `updateDisplay()`: Atualiza display conforme tela atual

**Finalidade**: Orquestrar todos os componentes do sistema IoT.

## Fluxo de Operação

### Inicialização (setup())

1. **Inicialização Serial**: Configuração da comunicação serial (115200 baud)
2. **Inicialização de Componentes**:
   - Display LCD
   - Sensor DHT11
   - Cliente MQTT
3. **Conexão WiFi**: 
   - Exibe animação de conexão
   - Mostra IP quando conectado
4. **Conexão MQTT**:
   - Conecta ao broker HiveMQ
   - Publica status "online"
   - Subscreve tópicos de controle
5. **Leitura Inicial**: Primeira leitura dos sensores
6. **Configuração de Telas**: Inicia rotação automática

### Loop Principal (loop())

1. **Manutenção MQTT**: Mantém conexão ativa
2. **Leitura de Sensores**: 
   - Sensor de luz (controla LED automaticamente)
   - Sensor DHT11 (temperatura/umidade)
3. **Gerenciamento de Telas**: Alterna telas a cada 5 segundos
4. **Atualização do Display**: Mostra dados da tela atual
5. **Verificação de Conexões**:
   - Reconecta WiFi se necessário
   - Reconecta MQTT se necessário
6. **Publicação de Dados**:
   - Dados de luz: quando LED muda de estado
   - Dados DHT: a cada 30 segundos
7. **Delay**: Pausa de 1 segundo entre iterações

## Telas do Display

### Tela 0: Sensor de Luz e LED
```
Luz:750
LED:OFF T:800
```
- Valor do sensor de luz
- Status do LED (ON/OFF)
- Threshold configurado

### Tela 1: Temperatura e Umidade
```
25.30 C
Humidade: 60.2%
```
- Temperatura em Celsius
- Umidade relativa em %

### Tela 2: Status MQTT
```
MQTT: OK
Msgs: 15
```
- Status da conexão MQTT
- Número de mensagens enviadas

## Comunicação MQTT

### Tópicos Utilizados

- **Dados de Luz**: `esp32/sensors/light`
- **Dados DHT**: `esp32/sensors/dht`
- **Status do Sistema**: `esp32/status`
- **Controle LED**: `esp32/control/led` (subscrição)

### Formato JSON dos Dados

**Dados de Luz**:
```json
{
  "light_value": 750,
  "led_state": false,
  "threshold": 800,
  "timestamp": 123456
}
```

**Dados DHT**:
```json
{
  "temperature": 25.30,
  "humidity": 60.2,
  "timestamp": 123456
}
```

**Status do Sistema**:
```json
{
  "status": "online",
  "client_id": "ESP32_IoT_001",
  "timestamp": 123456
}
```

## Controle Remoto

O sistema aceita comandos via MQTT no tópico `esp32/control/led`:
- `"on"`: Liga o LED manualmente
- `"off"`: Desliga o LED manualmente

## Configuração Necessária

Para usar este código, configure as seguintes variáveis:

```cpp
const char* Config::WIFI_SSID = "SuaRedeWiFi";
const char* Config::WIFI_PASSWORD = "SuaSenhaWiFi";
const char* Config::MQTT_BROKER = "broker.hivemq.com";
const char* Config::MQTT_CLIENT_ID = "ESP32_IoT_001";
const char* Config::MQTT_TOPIC_LIGHT = "esp32/sensors/light";
const char* Config::MQTT_TOPIC_DHT = "esp32/sensors/dht";
const char* Config::MQTT_TOPIC_STATUS = "esp32/status";
```

## Hardware Necessário

- ESP32 Development Board
- Sensor DHT11 (pino 2)
- Sensor de luz/fotorresistor (pino 32)
- LED (pino 19)
- Display LCD I2C 16x2 (endereço 0x27)
- Resistores e jumpers conforme necessário

## Funcionalidades Avançadas

- **Reconexão Automática**: WiFi e MQTT se reconectam automaticamente
- **Tratamento de Erros**: Validação de leituras dos sensores
- **Interface Dinâmica**: Alternância automática entre telas
- **Controle Remoto**: Recepção de comandos via MQTT
- **Monitoramento**: Contador de mensagens enviadas
- **Logs Detalhados**: Saída serial para debug

## Melhorias futuras

- Adicionar mais sensores
- Implementar configuração via web interface
- Armazenar dados em banco de dados
- Adicionar alertas por email/SMS
- Implementar OTA (Over-The-Air) updates
- Adicionar autenticação MQTT