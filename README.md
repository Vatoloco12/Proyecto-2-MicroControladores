# Proyecto-2-MicroControladores
#include <WiFi.h>

// Pines y constantes
const int pinSensor = 26; // GPIO26 (ADC0)
const float ganancia = 4.9;  // Ganancia del amplificador LM358
const float VREF = 3.3;      // Voltaje de referencia del ADC

// Wi-Fi: reemplaza con tus credenciales
const char* ssid = "INFINITUM085A";
const char* password = "Kexd9Rv3Ek";

WiFiServer server(80); // Puerto HTTP por defecto

void setup() {
  Serial.begin(115200);
  delay(1000);

  // Conectar a WiFi
  Serial.print("Conectando a WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\n✅ Conectado a WiFi");
  Serial.print("Dirección IP: ");
  Serial.println(WiFi.localIP());

  server.begin(); // Iniciar servidor web
}

void loop() {
  WiFiClient client = server.accept(); // Forma recomendada para aceptar conexiones
  if (client) {
    Serial.println("📡 Cliente conectado");
    while (client.connected()) {
      if (client.available()) {
        client.readStringUntil('\r');  // Leer solicitud HTTP
        client.read(); // Leer '\n'

        // Lectura del sensor
        int lecturaADC = analogRead(pinSensor);
        float voltajeAmplificado = lecturaADC * (VREF / 4095.0);
        float voltajeSensor = voltajeAmplificado / ganancia;
        float temperaturaC = voltajeSensor * 100.0;

        // Construir página web
        String html = "<!DOCTYPE html><html><head><meta charset='UTF-8'>";
        html += "<meta http-equiv='refresh' content='5'>";
        html += "<title>Temperatura Pico W</title></head><body>";
        html += "<h1>Temperatura actual:</h1>";
        html += "<p style='font-size:2em;'>" + String(temperaturaC, 2) + " &deg;C</p>";
        html += "</body></html>";

        // Enviar respuesta HTTP
        client.println("HTTP/1.1 200 OK");
        client.println("Content-Type: text/html");
        client.println("Connection: close");
        client.println();
        client.println(html);

        delay(10);
        break;
      }
    }
    client.stop();
    Serial.println("❌ Cliente desconectado");
  }
}
