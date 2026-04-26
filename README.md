# ESP32 + Arduino UNO Communication & Servo Control Project

This project contains two main tasks using ESP32 and Arduino UNO:
- UART communication with LED control
- WiFi Access Point servo motor control

---

# 📌 Task 1: ESP32 ↔ Arduino UNO Communication (UART + LED Control)

## 🎯 Objective
Send ON/OFF commands from ESP32 to Arduino UNO to control an LED.

---

## 🔧 Components Used
- ESP32 Dev Module
- Arduino UNO
- LED
- Resistor (220Ω or 330Ω)
- Breadboard and jumper wires

---

## 🔌 Wiring (Task 1)

ESP32 to Arduino:
- ESP32 GPIO17 (TX) → Arduino Pin 10 (RX SoftwareSerial)
- ESP32 GND → Arduino GND (common ground required)

LED on Arduino:
- Arduino Pin 8 → LED positive leg
- LED negative leg → GND through resistor

---

## 💻 ESP32 Code (Sender - ON/OFF Control)

```cpp
#define TX_PIN 17

void setup() {
  Serial.begin(115200);
  Serial2.begin(9600, SERIAL_8N1, -1, TX_PIN);

  Serial.println("Type ON or OFF");
}

void loop() {
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();

    if (cmd == "ON" || cmd == "OFF") {
      Serial2.println(cmd);
      Serial.print("Sent: ");
      Serial.println(cmd);
    } else {
      Serial.println("Only ON or OFF allowed");
    }
  }
}

```
## Arduino Code (Receiver + LED Control)
```cpp
#include <SoftwareSerial.h>

SoftwareSerial espSerial(10, 11);
const int ledPin = 8;

void setup() {
  Serial.begin(9600);
  espSerial.begin(9600);

  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);

  Serial.println("Arduino Ready");
}

void loop() {
  if (espSerial.available()) {
    String cmd = espSerial.readStringUntil('\n');
    cmd.trim();

    Serial.print("Received: ");
    Serial.println(cmd);

    if (cmd == "ON") {
      digitalWrite(ledPin, HIGH);
    }
    else if (cmd == "OFF") {
      digitalWrite(ledPin, LOW);
    }
  }
}
```
## TASK 2: ESP32 WIFI ACCESS POINT + SERVO CONTROL

# 🎯 Objective

Control a Servo Motor (SG90) using ESP32 Web Server (Access Point Mode)

## Control Buttons

- **Forward** → clockwise movement  
- **Backward** → counter-clockwise movement  

---

## 🔧 Components Used

- ESP32 Dev Module  
- Servo Motor (SG90)  
- Arduino UNO (used only for 5V power)  
- Jumper wires  

---

## 🔌 Wiring (Task 2)

### Servo Connections:

- **Servo Red (VCC)** → Arduino 5V  
- **Servo Brown/Black (GND)** → Arduino GND  
- **Servo Orange (Signal)** → ESP32 GPIO18  

---

## ⚠️ Important Notes

- Arduino GND must be connected to ESP32 GND  
- ESP32 is only controlling the signal (not powering the servo)  
- Arduino UNO is used only as a power source (5V)  

Important:

Arduino GND must be connected to ESP32 GND


```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <ESP32Servo.h>

Servo myServo;
WebServer server(80);

const char* ssid = "ESP32_SERVO";
const char* password = "12345678";

int servoPin = 18;

void handleRoot() {
  String page = "<html><body>";
  page += "<h2>ESP32 Servo Control</h2>";
  page += "<a href='/forward'><button>Forward</button></a><br><br>";
  page += "<a href='/backward'><button>Backward</button></a>";
  page += "</body></html>";

  server.send(200, "text/html", page);
}

void handleForward() {
  for (int i = 0; i <= 180; i++) {
    myServo.write(i);
    delay(10);
  }

  server.sendHeader("Location", "/");
  server.send(303);
}

void handleBackward() {
  for (int i = 180; i >= 0; i--) {
    myServo.write(i);
    delay(10);
  }

  server.sendHeader("Location", "/");
  server.send(303);
}

void setup() {
  Serial.begin(115200);

  myServo.attach(servoPin);
  myServo.write(90);

  WiFi.softAP(ssid, password);

  server.on("/", handleRoot);
  server.on("/forward", handleForward);
  server.on("/backward", handleBackward);

  server.begin();
}

void loop() {
  server.handleClient();
}
```
## NOTES

- ESP32 uses GPIO17 for UART TX in Task 1
- ESP32 uses GPIO18 for servo signal in Task 2
- Always connect GND between ESP32 and Arduino
- Arduino is only used for LED (Task 1) and 5V power (Task 2)
