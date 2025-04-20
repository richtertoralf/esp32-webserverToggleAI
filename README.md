# esp32-webserverToggleAI
mit einem ESP32-WROM-32 das AI Tracking einer PTZ-Kamera per Wifi ein- und ausschalten 
![IRremote](Steckbrett.jpg)

Zuerst  habe ich mit einem KY-022 Infrarot Empfänger Modul die Signale der PTZ-Fernbedienung ausgelesen.
Diese Signalsequenzen konnte ich dann nutzen, um das AI Tracking meiner HDKATOV PTZ-Kamera vom Smartphone aus per Wifi ein- und auszuschalten.

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <IRremote.hpp>

#define IR_SEND_PIN 4  // GPIO für IR-Senden (z. B. KY-005)

const char* ssid = "skitv";
const char* password = "xxxxxx";

WebServer server(80);

// === AI Toggle RAW-Daten (0x6A49) ===
uint16_t raw_ai_toggle[] = {
  2350,650, 1150,700, 500,650, 550,700,
  1100,700, 500,750, 450,700, 1150,650,
  550,700, 500,700, 1150,650, 550,650,
  1150,650, 550,650, 1150,700, 1100
};

void handleRoot() {
  String html = R"rawliteral(
    <!DOCTYPE html><html><head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>ESP32 AI Tracking</title>
    <style>
      body { font-family: sans-serif; text-align: center; margin-top: 50px; }
      button {
        font-size: 1.5rem; padding: 15px 30px;
        background-color: #28a745; color: white; border: none;
        border-radius: 8px; cursor: pointer;
      }
    </style>
    </head><body>
      <h2>AI Tracking Steuerung</h2>
      <form action="/toggle" method="POST">
        <button>AI TOGGLE SENDEN</button>
      </form>
    </body></html>
  )rawliteral";

  server.send(200, "text/html", html);
}

void handleToggle() {
  Serial.println("🟣 Web: Sende AI Toggle (5×)");
  for (int i = 0; i < 5; i++) {
    IrSender.sendRaw(raw_ai_toggle, sizeof(raw_ai_toggle) / sizeof(raw_ai_toggle[0]), 38);
    delay(100);
  }
  server.sendHeader("Location", "/");
  server.send(303); // redirect
}

void setup() {
  Serial.begin(115200);
  delay(200);

  IrSender.begin(IR_SEND_PIN, ENABLE_LED_FEEDBACK, USE_DEFAULT_FEEDBACK_LED_PIN);

  WiFi.begin(ssid, password);
  Serial.print("🔌 Verbinde mit WLAN ");
  Serial.println(ssid);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.print("✅ Verbunden! IP-Adresse: ");
  Serial.println(WiFi.localIP());

  server.on("/", handleRoot);
  server.on("/toggle", HTTP_POST, handleToggle);
  server.begin();
  Serial.println("🌍 Webserver gestartet");
}

void loop() {
  server.handleClient();
}
```
Ausgabe im Serial Monitor:
```
19:57:25.565 -> ✅ Verbunden! IP-Adresse: 192.168.95.115
19:57:25.565 -> 🌍 Webserver gestartet
20:02:17.468 -> 🟣 Web: Sende AI Toggle (5×)
20:03:13.617 -> 🟣 Web: Sende AI Toggle (5×)
20:03:29.604 -> 🟣 Web: Sende AI Toggle (5×)
```
