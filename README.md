# Practica-8
-En este repositorio vamos a combinar el programa NodeRed y Wokwi

## Introducción
-Realizaremso una conexion a un servidor utilizando NodeRed y Wokwi

### Descripción
-En esta practica realizaremos la conexion desde el simulador Node Red y Wokwi para poder asi reflejar en un tarjeta  Esp32  y utilizando un sensor DHT22 para la obtención de datos de temperatura y humedad;


Material Necesario
Necesitamos abrir ambos simuladores ;
-Node.js-  ;Para ejecutar abrir CMD colocar "Node.js" en el comando e ingresar en linea a http://localhost:1880/
-WOKWI ; Dentro del simulador utilizaremos lo siguiente;
     1. Tarjeta ESP 32
     2. Sensor DHT22

-  1°- En WOWKI colocaremos el siguiente Codigo;
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
const char* mqtt_server = "3.65.168.153";
String username_mqtt="ACH1309";
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
    client.publish("Cisneros1", output.c_str());
  }
}


- 2°-Instalamos las librerias que utilizaremos
- ![]()
  

- 3°- Agregar el Sensor DHT22 y realizar las conexiones con la ESP22
- ![]()

- 4° - En el programa Node Red vamos a instalar  el bloque de mqtt in, posteriormente bloque json , asi mismo las function y los bloques de Chart y Gaugue Correspondientes 
  -En bloque mqtt in colocamos el nombre del topico y el numero de la IP que se utilizara (esta dede de ser la misma que la que se coloco en el codigo)
  -El bloque json colocar la accion de Always convert to JavaScript Object
  -En los bloques function colocaremos estos codigos uno en cada uno ;
    msg.payload = msg.payload.TEMPERATURA;
    msg.topic = "TEMPERATURA";
    return msg;
  
    msg.payload = msg.payload.HUMEDAD;
    msg.topic = "HUMEDAD";
    return msg;
  -En bloques de Chart y Gaugue
   Selecionar los grupos de chart en graficos y los gaugue en indicador, colocar los datos que corresponden 
![]()
![]()
![]()
![]()



-5° - Hacemos correr la programacion en WOKWI asegurandonos que se conecto de manera correcta. Finalmente el  NODE-RED y la ESP32 dentro de WOKWI arrojaran los resultados deseados
![]()

- 

