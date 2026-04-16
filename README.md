# smart-hydroponics-esp32-s2
# 🌱 Smart Hydroponics System (ESP32-S2)

An IoT-based hydroponics automation system built with ESP32-S2 (Franzininho) that monitors environmental conditions and automatically controls plant care.

---

## 🚀 Features

* 🌡 Temperature Monitoring (DHT22)
* 💧 Humidity Monitoring
* 🧪 pH Monitoring (analog simulation)
* 🚿 Automatic Irrigation System
* 💨 Fan Control (temperature-based)
* ⚗️ Chemical Dosing (pH control)
* 🖥 LCD Real-Time Display
* 🌐 Web Dashboard (WiFi)
* 🔴 LED Indicators + Relay Control

---

## 📸 Demo

👉 Add your demo GIF here:

![demo](demo/demo.gif)

---

## 🧠 System Logic

| Condition   | Action           |
| ----------- | ---------------- |
| Temp > 28°C | Fan ON           |
| pH > 6.5    | Chemical Pump ON |
| Soil Dry    | Water Pump ON    |

---

## 🔌 Hardware Components

* ESP32-S2 (Franzininho)
* DHT22 Sensor
* Potentiometer (pH simulation)
* Slide Switch (soil simulation)
* 3x Relay Modules
* 3x LEDs + Resistors
* LCD 16x2 I2C

---

## 🔧 Pin Configuration

```
GPIO4   -> DHT22
GPIO7   -> pH sensor
GPIO35  -> soil switch

GPIO12  -> FAN relay + LED
GPIO13  -> PH relay + LED
GPIO14  -> WATER relay + LED

GPIO8   -> LCD SDA
GPIO9   -> LCD SCL
```

---

## 🌐 Wokwi Simulation

👉 https://wokwi.com/projects/461370076504628225

---

## 📊 Web Interface

* Real-time sensor monitoring
* Live system status
* Automatic refresh

---

## 📂 Project Structure

```
.
├── code/
│   └── main.ino
├── demo/
│   ├── demo.gif
│   └── video.mp4
├── images/
│   └── circuit.png
└── README.md
```

---

## 🚀 Future Improvements

* 📱 Mobile App Control
* ☁️ Firebase Integration
* 📊 Real-time Graphs
* 🔔 Notifications (Telegram)

---

## 👨‍💻 Author

Your Name
Electrical & Computer Engineering Student

---

## ⭐ If you like this project, give it a star!
