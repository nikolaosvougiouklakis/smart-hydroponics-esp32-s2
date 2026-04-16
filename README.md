#include <WiFi.h>
#include <WebServer.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include "DHT.h"

// ---------------- PINS ----------------
#define DHTPIN 4
#define DHTTYPE DHT22

#define PH_PIN 7
#define SOIL_PIN 35

#define FAN_PIN 12
#define PH_PUMP_PIN 13
#define WATER_PUMP_PIN 14

#define I2C_SDA 8
#define I2C_SCL 9

// ---------------- WIFI ----------------
const char* WIFI_SSID = "Wokwi-GUEST";
const char* WIFI_PASS = "";

// ---------------- OBJECTS ----------------
DHT dht(DHTPIN, DHTTYPE);
WebServer server(80);
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ---------------- VARIABLES ----------------
float temp = 0.0;
float hum = 0.0;
float ph = 0.0;

String tempStatus = "OK";
String humStatus = "OK";
String phStatus = "OK";
String actionText = "SYSTEM OK";

bool fanOn = false;
bool phPumpOn = false;
bool waterOn = false;
bool wifiReady = false;

unsigned long lastRead = 0;
unsigned long lastLcdChange = 0;
unsigned long lastWifiRetry = 0;
bool secondScreen = false;

// ---------------- HELPERS ----------------
String activeOutputs() {
  String s = "";
  if (fanOn) s += "FAN ";
  if (phPumpOn) s += "PH ";
  if (waterOn) s += "WATER ";
  if (s == "") s = "NONE";
  return s;
}

void connectWiFiOnce() {
  Serial.println("Connecting WiFi...");
  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASS);

  unsigned long start = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - start < 8000) {
    delay(250);
    Serial.print(".");
  }
  Serial.println();

  if (WiFi.status() == WL_CONNECTED) {
    wifiReady = true;
    Serial.println("WiFi connected!");
    Serial.print("IP: ");
    Serial.println(WiFi.localIP());

    server.on("/", []() {
      String html = "<!DOCTYPE html><html><head>";
      html += "<meta charset='UTF-8'>";
      html += "<meta http-equiv='refresh' content='2'>";
      html += "<title>Smart Farm</title>";
      html += "</head><body>";
      html += "<h2>Smart Hydroponics</h2>";
      html += "<p>Temperature: " + String(temp, 1) + " C (" + tempStatus + ")</p>";
      html += "<p>Humidity: " + String(hum, 1) + " % (" + humStatus + ")</p>";
      html += "<p>pH: " + String(ph, 2) + " (" + phStatus + ")</p>";
      html += "<p>Action: " + actionText + "</p>";
      html += "<p>Outputs: " + activeOutputs() + "</p>";
      html += "</body></html>";
      server.send(200, "text/html; charset=UTF-8", html);
    });

    server.on("/data", []() {
      String json = "{";
      json += "\"temp\":" + String(temp, 1) + ",";
      json += "\"hum\":" + String(hum, 1) + ",";
      json += "\"ph\":" + String(ph, 2) + ",";
      json += "\"tempStatus\":\"" + tempStatus + "\",";
      json += "\"humStatus\":\"" + humStatus + "\",";
      json += "\"phStatus\":\"" + phStatus + "\",";
      json += "\"action\":\"" + actionText + "\",";
      json += "\"outputs\":\"" + activeOutputs() + "\"";
      json += "}";
      server.send(200, "application/json", json);
    });

    server.begin();
    Serial.println("HTTP server started");
  } else {
    wifiReady = false;
    Serial.println("WiFi not connected - system will continue without web");
  }
}

void retryWiFiIfNeeded() {
  if (wifiReady) return;
  if (millis() - lastWifiRetry < 10000) return;
  lastWifiRetry = millis();
  connectWiFiOnce();
}

void updateStatuses() {
  if (temp > 28.0) tempStatus = "HIGH";
  else if (temp < 18.0) tempStatus = "LOW";
  else tempStatus = "OK";

  if (hum < 40.0) humStatus = "LOW";
  else if (hum > 80.0) humStatus = "HIGH";
  else humStatus = "OK";

  if (ph < 5.5) phStatus = "ACID";
  else if (ph > 6.5) phStatus = "ALK";
  else phStatus = "OK";
}

void controlOutputs() {
  fanOn = (temp > 28.0);
  phPumpOn = (ph > 6.5);

  int soilState = digitalRead(SOIL_PIN); // LOW = dry soil
  waterOn = (soilState == LOW);

  digitalWrite(FAN_PIN, fanOn ? HIGH : LOW);
  digitalWrite(PH_PUMP_PIN, phPumpOn ? HIGH : LOW);
  digitalWrite(WATER_PUMP_PIN, waterOn ? HIGH : LOW);

  if (waterOn) actionText = "WATERING";
  else if (phPumpOn) actionText = "PH FIX";
  else if (fanOn) actionText = "COOLING";
  else actionText = "SYSTEM OK";
}

void updateLCD() {
  if (millis() - lastLcdChange > 3000) {
    lastLcdChange = millis();
    secondScreen = !secondScreen;
  }

  lcd.setCursor(0, 0);
  lcd.print("                ");
  lcd.setCursor(0, 1);
  lcd.print("                ");

  if (!secondScreen) {
    lcd.setCursor(0, 0);
    lcd.print("T:");
    lcd.print(temp, 1);
    lcd.print(" H:");
    lcd.print(hum, 0);

    lcd.setCursor(0, 1);
    lcd.print("pH:");
    lcd.print(ph, 1);
    lcd.print(" ");
    lcd.print(actionText.substring(0, min((int)actionText.length(), 8)));
  } else {
    lcd.setCursor(0, 0);
    lcd.print("T:");
    lcd.print(tempStatus);
    lcd.print(" H:");
    lcd.print(humStatus);

    lcd.setCursor(0, 1);
    lcd.print("pH:");
    lcd.print(phStatus);
    lcd.print(" ");
    if (waterOn) lcd.print("WATER");
    else if (phPumpOn) lcd.print("PUMP");
    else if (fanOn) lcd.print("FAN");
    else lcd.print("IDLE");
  }
}

// ---------------- SETUP ----------------
void setup() {
  Serial.begin(115200);
  delay(300);

  Wire.begin(I2C_SDA, I2C_SCL);
  dht.begin();

  lcd.init();
  lcd.backlight();

  pinMode(FAN_PIN, OUTPUT);
  pinMode(PH_PUMP_PIN, OUTPUT);
  pinMode(WATER_PUMP_PIN, OUTPUT);
  pinMode(SOIL_PIN, INPUT);

  digitalWrite(FAN_PIN, LOW);
  digitalWrite(PH_PUMP_PIN, LOW);
  digitalWrite(WATER_PUMP_PIN, LOW);

  lcd.setCursor(0, 0);
  lcd.print("Smart Farm");
  lcd.setCursor(0, 1);
  lcd.print("Booting...");
  
  connectWiFiOnce();
}

// ---------------- LOOP ----------------
void loop() {
  if (wifiReady) {
    server.handleClient();
  } else {
    retryWiFiIfNeeded();
  }

  if (millis() - lastRead >= 2000) {
    lastRead = millis();

    float newTemp = dht.readTemperature();
    float newHum = dht.readHumidity();

    if (!isnan(newTemp) && !isnan(newHum)) {
      temp = newTemp;
      hum = newHum;
    }

    int raw = analogRead(PH_PIN);
    ph = raw * (14.0 / 4095.0);

    updateStatuses();
    controlOutputs();

    Serial.printf("T: %.1f (%s) | H: %.1f (%s) | pH: %.2f (%s) | ACTION: %s | OUT: %s\n",
      temp, tempStatus.c_str(),
      hum, humStatus.c_str(),
      ph, phStatus.c_str(),
      actionText.c_str(),
      activeOutputs().c_str()
    );
  }

  updateLCD();
}
