# LOJA DE VINHOS

Este projeto cria o banco de dados Loja de Vinhos utilizando o MySQL. O objetivo é armazenar informações sobre vinhos, vinícolas e as regiões a que pertencem. As tarefas envolvem a criação do modelo físico, a inserção de registros, a consulta de dados, e a configuração de um usuário com permissões limitadas.

## 1. Configuração do ESP32 no WokWi
```bash
Acesse o WokWi e escolha o ESP32 como microcontrolador.
Configure os sensores/atuadores no ambiente de simulação:
Se você escolher um sensor de temperatura, adicione o sensor DHT22.
Se for um LED, adicione-o para simular a iluminação inteligente.
Configure o código em MicroPython para leitura dos sensores e controle dos atuadores.
```

## 2. Programação em MicroPython
```bash
import network
from umqttsimple import MQTTClient
import time
import dht
from machine import Pin

# Configurações de Wi-Fi
nome_wifi = 'seu_ssid_wifi'
senha_wifi = 'sua_senha_wifi'

# Configurações do Broker MQTT
servidor_mqtt = 'broker.hivemq.com'
id_cliente = 'ESP32'
topico_publicacao_temp = 'cidade/inteligente/temperatura'
topico_publicacao_umid = 'cidade/inteligente/umidade'

# Configuração do DHT11
sensor_dht = dht.DHT11(Pin(15))  # Define o sensor no pino 15 do ESP32

def conectar_wifi():
    estacao = network.WLAN(network.STA_IF)
    estacao.active(True)
    estacao.connect(nome_wifi, senha_wifi)

    while not estacao.isconnected():
        time.sleep(1)

def conectar_e_assinar():
    cliente = MQTTClient(id_cliente, servidor_mqtt)
    cliente.connect()
    return cliente

def principal():
    conectar_wifi()
    cliente = conectar_e_assinar()

    while True:
        try:
            # Leitura de temperatura e umidade do DHT11
            sensor_dht.measure()
            temperatura = sensor_dht.temperature()
            umidade = sensor_dht.humidity()

            # Publicar os dados no MQTT
            cliente.publish(topico_publicacao_temp, str(temperatura))
            cliente.publish(topico_publicacao_umid, str(umidade))

            print(f"Temperatura: {temperatura}°C, Umidade: {umidade}%")

        except OSError as e:
            print("Erro ao ler o sensor DHT11.")

        time.sleep(10)

if __name__ == "__main__":
    principal()

```

## 3. Integração dos Sensores
```bash
Sensores e Atuadores Utilizados:
DHT11: Leitura de temperatura e umidade.
LED: Controle de iluminação inteligente ou sinalização.
Funções dos Sensores e Atuadores no Contexto de Smart City:

DHT11: Pode ser usado para monitoramento ambiental em estufas ou sistemas de controle climático.
LED: Simulação de iluminação pública inteligente, onde o LED pode ser ligado ou desligado remotamente com base em condições ou comandos manuais.
```

## 4. Conectividade e Controle Remoto via MQTT

```bash
Acesse o HiveMQ WebSocket Client.
Conecte-se ao broker inserindo broker.hivemq.com no campo Host e clique em Connect.
Assine os tópicos que você configurou no código do ESP32 para receber as leituras de temperatura e umidade:
Topic to subscribe: cidade/inteligente/temperatura
Topic to subscribe: cidade/inteligente/umidade
Para controlar o LED, publique no tópico de controle:
Topic to publish: cidade/inteligente/atuador
Message: ligar_led ou desligar_led.
```

## 5. Dashboard no Node-RED
```bash
Instale o Node-RED em sua máquina local ou use uma instância hospedada.
Abra o editor do Node-RED e configure os nós MQTT para assinar os tópicos de temperatura, umidade e atuador.

Adicionar um nó de entrada MQTT:

Conecte-se ao broker broker.hivemq.com.
Assine o tópico cidade/inteligente/temperatura e cidade/inteligente/umidade para visualizar os dados.
Visualização no Dashboard:

Adicione gráficos para visualizar as leituras de temperatura e umidade em tempo real.
Para controle, adicione um botão para enviar comandos MQTT ao tópico cidade/inteligente/atuador para ligar ou desligar o LED.
Exemplo de Configuração no Node-RED:
Gráfico de Temperatura:
Usar o nó MQTT In para se inscrever no tópico cidade/inteligente/temperatura.
Conectar a um nó de gráfico para exibir as leituras de temperatura ao longo do tempo.
Botão de Controle de LED:
Usar o nó Button no dashboard, e quando clicado, ele deve publicar a mensagem ligar_led ou desligar_led no tópico cidade/inteligente/atuador.
```

