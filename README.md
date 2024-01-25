# PR-CTICA-8

Dentro de este repositorio se muestra la manera de programar una ESP32 con el sensor DHT11 y además, que los datos obtenidos se vean reflejados en el programa Node-RED a través de una conexión WIFI.

INTRODUCCIÓN

La Esp32 la utilizamos en un entorno de adquision de datos, lo cual en esta practica ocuparemos un sensor (DTH11) para adquirir temperatura y humedad del entorno, tambien utilizaremos el programa Node-red para trabajar de manera visual y esquematica la programación grafica del flujo de la información, la cual podremos observar los resultados en la interfaz de Node-Red; cabe aclarar que en esta practica se usara un simulador llamado WOKWI y un programa llamado Node-RED.

REQUISITOS

Para poder usar este repositorio necesitas entrar a la plataforma WOKWI y tener instalado correctamente el programa Node-RED.

INSTRUCCIONES

1- Abrir la terminal de programación y colocar la siguente programación:

PROGRAMACIÓN DE ESP32

#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "52.57.167.175";
String username_mqtt="educatronicosiot";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {


delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("Diplomado", output.c_str());
  }
}

2- Instalar las librerias de DHT sensor library for ESPx, PubSubClient y ArduinoJson.

3- Hacer la conexion de DHT11 con la ESP32.

![Captura de pantalla 2024-01-20 084422](https://github.com/robertopatino42/PR-CTICA-8/assets/153964688/bf08a6a9-e467-4980-89bc-0293b68ed27e)


4- Cambiando al programa de Node-RED primero tendran que colocar el bloque de mqtt in, posteriormente los siguientes bloques.

![Captura de pantalla 2024-01-20 084320](https://github.com/robertopatino42/PR-CTICA-8/assets/153964688/02c32cde-6e80-4ae4-b63c-f7304cde12e1)

5- La conexiónfinal del NODE-RED y la ESP32 quedaría de esta manera.

![Captura de pantalla 2024-01-20 084520](https://github.com/robertopatino42/PR-CTICA-8/assets/153964688/dbb4c1f3-36b0-4715-a1af-889eca8fce23)





