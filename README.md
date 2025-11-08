# 🏆 Projeto IoT – Sistema de Monitoramento de Partida de Futebol Feminino

### 📡 ESP32 • MQTT • Dashboard em Python • Passa a Bola

---

## 🎯 **Visão Geral do Projeto**

Este projeto IoT foi desenvolvido para monitorar eventos importantes de uma partida de futebol feminino em **tempo real**, utilizando um **ESP32**, comunicação via **MQTT** e um **dashboard profissional em Python**.

Ele foi criado para atender a identidade visual e as necessidades do canal **Passa a Bola**, das criadoras **Luana Maluf** e **Ale Xavier**, oferecendo uma ferramenta prática, moderna e eficiente para transmissões esportivas.

---

# Link do video de apresentação do projeto
https://youtu.be/4LO2tyIfdLQ

---

## 🚀 **Objetivo do Sistema**

O sistema permite registrar e exibir ao vivo:

* ✅ Gols do Time A
* ✅ Gols do Time B
* ✅ Cartões Amarelos
* ✅ Cartões Vermelhos

Tudo isso através de botões físicos conectados ao ESP32, que envia os dados instantaneamente para o dashboard via MQTT.

---

## 🧩 **Arquitetura Resumida**

```
[ Botões ESP32 ] → [ MQTT Broker ] → [ Dashboard Python ] → [ Navegador ]
```

* O ESP32 publica mensagens em tópicos específicos
* O Dashboard assina esses tópicos e atualiza a interface automaticamente
* O usuário visualiza tudo em tempo real em um layout estilizado

---

## 📝 Esquemático de Ligação
<img width="1919" height="971" alt="Captura de tela 2025-11-07 214926" src="https://github.com/user-attachments/assets/56bf573e-78f3-4b62-9042-43aff9b5528f" />

---

## ⚙️ Configuração do Ambiente
1. Instalação do Arduino IDE

- Baixar e instalar Arduino IDE.

- Adicionar suporte ao ESP32:

  - Arquivo > Preferências > URL adicionais de placas:

      https://dl.espressif.com/dl/package_esp32_index.json


  - Ferramentas > Placa > Gerenciador de Placas → Pesquisar e instalar ESP32.

2. Bibliotecas Necessárias

Instale as seguintes bibliotecas via Gerenciador de Bibliotecas:

- DHT sensor library (by Adafruit)

- WiFi

- PubSubClient (para MQTT)

---

## 📲 Configuração do MyMQTT
O ESP32 **publica** mensagens nesses tópicos sempre que um botão é pressionado.
O dashboard em Python **assina** esses mesmos tópicos e atualiza a interface em tempo real.


Por ser um broker público, não é necessário criar conta ou gerar token. Basta conectar com o endereço e porta acima.

---

# 🔧 **Código Fonte – ESP32**
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
// Integrantes do grupo:
// Guilherme Eduardo de Lima
// Guilherme de Paula
// Enzo de Faria
// Matheus Gomes

// WiFi
const char* ssid = "Wokwi-GUEST";
const char* password = "";

// MQTT Broker
const char* mqtt_server = "broker.hivemq.com";
WiFiClient espClient;
PubSubClient client(espClient);

// Botões
const int btnGolA = 13;
const int btnGolB = 12;
const int btnAmarelo = 14;
const int btnVermelho = 27;

bool lastStateGolA = HIGH;
bool lastStateGolB = HIGH;
bool lastStateAmarelo = HIGH;
bool lastStateVermelho = HIGH;

void setup_wifi() {
  Serial.println("Conectando ao WiFi...");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\n✅ WiFi conectado!");
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Conectando MQTT...");
    if (client.connect("ESP32_Futebol")) {
      Serial.println("✅ conectado!");
    } else {
      Serial.print("❌ falhou, rc=");
      Serial.print(client.state());
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(btnGolA, INPUT_PULLUP);
  pinMode(btnGolB, INPUT_PULLUP);
  pinMode(btnAmarelo, INPUT_PULLUP);
  pinMode(btnVermelho, INPUT_PULLUP);

  setup_wifi();
  client.setServer(mqtt_server, 1883);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  if (lastStateGolA == HIGH && digitalRead(btnGolA) == LOW) {
    client.publish("futebol/golA", "1");
    Serial.println("⚽ Gol Time A!");
  }
  lastStateGolA = digitalRead(btnGolA);

  if (lastStateGolB == HIGH && digitalRead(btnGolB) == LOW) {
    client.publish("futebol/golB", "1");
    Serial.println("⚽ Gol Time B!");
  }
  lastStateGolB = digitalRead(btnGolB);

  if (lastStateAmarelo == HIGH && digitalRead(btnAmarelo) == LOW) {
    client.publish("futebol/cartaoAmarelo", "1");
    Serial.println("🟨 Cartão Amarelo!");
  }
  lastStateAmarelo = digitalRead(btnAmarelo);

  if (lastStateVermelho == HIGH && digitalRead(btnVermelho) == LOW) {
    client.publish("futebol/cartaoVermelho", "1");
    Serial.println("🟥 Cartão Vermelho!");
  }
  lastStateVermelho = digitalRead(btnVermelho);

  delay(10);
}
```

---

## 📊 Dashboard em Python (Monitoramento em Tempo Real)
Para complementar o projeto e permitir a visualização dos dados recebidos tempo real, foi desenvolvido um Dashboard interativo utilizando Python, com as bibliotecas Dash e MQTT.

Este painel exibe um placar com as informações de gols dos dois times e o numero de cartões aplicados no jogo, amarelos e vermelhos, a partir das mensagens recebidas via broker MQTT.

---
## 📝 DASHBOARD
<img width="1919" height="958" alt="Captura de tela 2025-11-07 204117" src="https://github.com/user-attachments/assets/8e69869d-ef8b-48ff-83bd-e5db303ec859" />

---

# 🖥️ **Código Fonte – Dashboard em Python (Dash / MQTT)**

```python
# Integrantes do grupo:
# Guilherme Eduardo de Lima
# Guilherme de Paula
# Enzo de Faria
# Matheus Gomes

import dash
from dash import dcc, html
import dash_bootstrap_components as dbc
from dash.dependencies import Output, Input
import paho.mqtt.client as mqtt
import threading
import datetime

# ---------- CONFIGURAÇÕES MQTT ----------
MQTT_BROKER = "broker.hivemq.com"
MQTT_PORT = 1883
TOPICS = [
    "futebol/golA",
    "futebol/golB",
    "futebol/cartaoAmarelo",
    "futebol/cartaoVermelho"
]

# ---------- VARIÁVEIS ----------
gols_A = 0
gols_B = 0
amarelos = 0
vermelhos = 0
last_update = "Aguardando eventos..."

# ---------- CALLBACK MQTT ----------
def on_connect(client, userdata, flags, rc):
    print("Conectado ao MQTT:", rc)
    for t in TOPICS:
        client.subscribe(t)

def on_message(client, userdata, msg):
    global gols_A, gols_B, amarelos, vermelhos, last_update

    timestamp = datetime.datetime.now().strftime("%H:%M:%S")
    last_update = timestamp

    if msg.topic == "futebol/golA":
        gols_A += 1
    elif msg.topic == "futebol/golB":
        gols_B += 1
    elif msg.topic == "futebol/cartaoAmarelo":
        amarelos += 1
    elif msg.topic == "futebol/cartaoVermelho":
        vermelhos += 1

    print("✅ Evento recebido:", msg.topic, "|", timestamp)

# ---------- THREAD MQTT ----------
def mqtt_thread():
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect(MQTT_BROKER, MQTT_PORT, 60)
    client.loop_forever()

threading.Thread(target=mqtt_thread, daemon=True).start()

# ---------- CORES ----------
COR_FUNDO = "#FFFFFF"
VERDE = "#5E8A2D"
ROXO_ESCURO = "#512A5E"
ROXO_ROSADO = "#A55378"

# ---------- DASHBOARD ----------
app = dash.Dash(__name__, external_stylesheets=[dbc.themes.FLATLY])
app.title = "Passa Bola — Futebol Feminino"

# ---------- BANNER ----------
banner = dbc.Container(
    dbc.Row([
        dbc.Col(
            html.Img(
                src="/assets/logo.png",
                style={"height": "120px", "width": "120px", "objectFit": "contain", "marginLeft": "10px"}
            ),
            width="auto",
            style={"display": "flex", "alignItems": "center"}
        ),
        dbc.Col(
            html.Div([
                html.H1(
                    "PASSA A BOLA - FUTEBOL FEMININO⚽",
                    style={"textAlign": "center", "color": ROXO_ESCURO, "fontWeight": "bold", "fontSize": "58px"}
                ),
            ]),
            width=True,
            style={"display": "flex", "justifyContent": "center", "alignItems": "center"}
        ),
        dbc.Col(width="auto"),
    ], align="center"),
    fluid=True
)

# ---------- PLACAR ----------
placar = dbc.Container(
    dbc.Row([
        dbc.Col(html.Div([
            html.H2("TIME A", style={"textAlign": "center", "color": ROXO_ESCURO}),
            html.H1(id="golsA", style={"fontSize": "130px", "textAlign": "center", "fontWeight": "bold", "color": ROXO_ROSADO})
        ]), md=4),

        dbc.Col(html.Div([
            html.H1("X", style={"textAlign": "center", "fontSize": "110px", "color": VERDE, "fontWeight": "bold"})
        ]), md=2),

        dbc.Col(html.Div([
            html.H2("TIME B", style={"textAlign": "center", "color": ROXO_ESCURO}),
            html.H1(id="golsB", style={"fontSize": "130px", "textAlign": "center", "fontWeight": "bold", "color": ROXO_ROSADO})
        ]), md=4),
    ], justify="center"),
    fluid=True
)

# ---------- CARTÕES ----------
cartoes = dbc.Container([
    dbc.Row([
        dbc.Col(dbc.Card([
            dbc.CardHeader("Cartões Amarelos", style={"backgroundColor": ROXO_ESCURO, "color": "white", "fontWeight": "bold"}),
            dbc.CardBody(html.H1(id="amarelos", style={"textAlign": "center", "color": ROXO_ROSADO}))
        ], style={"borderRadius": "18px", "border": f"3px solid {VERDE}"}), md=3),

        dbc.Col(dbc.Card([
            dbc.CardHeader("Cartões Vermelhos", style={"backgroundColor": ROXO_ESCURO, "color": "white", "fontWeight": "bold"}),
            dbc.CardBody(html.H1(id="vermelhos", style={"textAlign": "center", "color": ROXO_ROSADO}))
        ], style={"borderRadius": "18px", "border": f"3px solid {VERDE}"}), md=3),
    ], justify="center")
], fluid=True)

# ---------- ÚLTIMO EVENTO ----------
ultimo_evento = dbc.Container([
    html.Br(),
    html.H4("📅 Último Evento Recebido:", style={"textAlign": "center", "color": ROXO_ESCURO}),
    html

```
## 💻 Tecnologias Utilizadas

Dash → Criação da interface e dos gráficos interativos.

Dash Bootstrap Components → Estilização com tema escuro e design responsivo.

Paho-MQTT → Comunicação com o broker MQTT em tempo real.

Threading e JSON → Processamento paralelo das mensagens e estruturação dos dados recebidos.

## 🔌 Execução do Dashboard

Certifique-se de ter o Python 3 instalado no sistema.

Instale as dependências necessárias executando no terminal:

pip install dash plotly dash-bootstrap-components paho-mqtt


Verifique se o ESP32 já está publicando dados no broker MQTT.

Salve o arquivo do dashboard como dashboard.py.

Execute o dashboard com o comando:

python dashboard.py


Após a inicialização, o terminal exibirá uma mensagem semelhante a:

Running on http://127.0.0.1:8050/


Acesse o endereço no navegador (geralmente http://localhost:8050) para visualizar o dashboard em tempo real.

---

## 🧪 Testes Realizados

- ✅ Conexão WiFi estável no ESP32.

- ✅ Publicação periódica de dados dos sensores no broker MQTT.

- ✅ Visualização dos valores no app MyMQTT em tempo real.

---

## 📚 Referências
- ESP32 — Documentação oficial: https://www.espressif.com

---

# 🏁 Conclusão
Este projeto demonstra de forma prática como:

- Integrar componentes com um microcontrolador ESP32;

- Utilizar protocolos IoT (MQTT) para comunicação em tempo real;

- Visualizar dados e atuar remotamente usando um aplicativo móvel.

A base desenvolvida pode ser facilmente expandida para incluir estatísticas, tempo de jogo, substituições e muito mais.

---

## 👥 Projeto IoT desenvolvido por:
  ## Guilherme Eduardo de Lima
  ## Guilherme de Paula
  ## Enzo de Faria
  ## Matheus Gomes




