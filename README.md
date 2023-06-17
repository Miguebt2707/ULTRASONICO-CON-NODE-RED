# Conexion-ESP32-Con-Node-Red

 LO QUE SE HIZO EN ESTA PRACTICA FUE EL REALIZAR UNA CONEXION DIRECTAMENTE EN LA RED PARA QUE LOGRARAN INTERACTUAR SENSORES QUE SE TIENEN EN UNA CONEXION Y VER LOS DATOS QUE ARROJAN EN UNOS DISPOSITIVOS PARECIDOS A UNOS TACOMETROS EN UNA INTERFAS EN INTERNET, PARA ESO SE TUVO QUE CONSEGUIR LA DIRECCION DE LA TERMINAL DE LA CUAL IBA A SALIR Y A LA QUE IBA A ENTRAR
 HACIENDO COMO PRIMER PASO LA CONEXION EN "WOKWI DE LOS DISPOSITIVOS QUE UTILIZAMOS, LOS CUALER FUERON:
 
## LOS MATERIALES FUERON:
  - ESP32
  - DTH22

![](https://raw.githubusercontent.com/Miguebt2707/PRACTICA-NIVEL-DE-AGUA/e500a7ab6a1891e21af34150646785fbe6ba026a/Captura%20de%20pantalla%202023-06-10%201147441.png)

![](https://raw.githubusercontent.com/Miguebt2707/PRACTICA-NIVEL-DE-AGUA/e500a7ab6a1891e21af34150646785fbe6ba026a/Captura%20de%20pantalla%202023-06-10%201148262.png)

![](https://raw.githubusercontent.com/Miguebt2707/PRACTICA-NIVEL-DE-AGUA/e500a7ab6a1891e21af34150646785fbe6ba026a/Captura%20de%20pantalla%202023-06-10%20114923.png)

![](https://raw.githubusercontent.com/Miguebt2707/PRACTICA-NIVEL-DE-AGUA/e500a7ab6a1891e21af34150646785fbe6ba026a/Captura%20de%20pantalla%202023-06-10%201150154.png)

![](https://raw.githubusercontent.com/Miguebt2707/PRACTICA-NIVEL-DE-AGUA/e500a7ab6a1891e21af34150646785fbe6ba026a/Captura%20de%20pantalla%202023-06-10%20115055.png)


```
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
const char* mqtt_server = "44.195.202.69";
String username_mqtt="Miguelon";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;
const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12;   //Pin digital 3 para el Echo del sensor
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
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm

TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 3000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    //doc["TEMPERATURA"] = String(data.temperature, 1);
    //doc["HUMEDAD"] = String(data.humidity, 1);
   doc [ "DISTANCIA" ] =d;

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("MigueBT2707", output.c_str());
  }
}
```