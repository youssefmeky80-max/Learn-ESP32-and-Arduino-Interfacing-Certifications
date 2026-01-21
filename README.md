#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

// ---------- WiFi ----------
#define WIFI_SSID "Orange-mariem"
#define WIFI_PASSWORD "RzdCYhzR"

// ---------- Firebase REST URL ----------
#define FIREBASE_URL "https://mind-development-6d446-default-rtdb.firebaseio.com/test.json?auth=PUT_YOUR_DATABASE_SECRET_HERE"

// ---------- Pins ----------
#define LED1_PIN 2
#define LED2_PIN 4
#define MOTION_PIN 19
#define GAS_PIN 34

void setup() {
  Serial.begin(115200);

  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(MOTION_PIN, INPUT);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Connected!");
}

void loop() {
  int motion = digitalRead(MOTION_PIN);
  int gasValue = analogRead(GAS_PIN);

  // -------- إرسال البيانات --------
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    // -------- إرسال بيانات --------
    StaticJsonDocument<200> jsonDoc;
    jsonDoc["motion"] = motion;
    jsonDoc["gas"] = gasValue;

    String requestBody;
    serializeJson(jsonDoc, requestBody);

    http.begin(FIREBASE_URL);
    http.addHeader("Content-Type", "application/json");
    int httpResponseCode = http.PUT(requestBody);

    if (httpResponseCode > 0) {
      Serial.println("Data sent successfully");
    } else {
      Serial.println("Failed to send data");
    }
    http.end();

    // -------- قراءة بيانات LED --------
    http.begin(FIREBASE_URL);
    int code = http.GET();
    if (code > 0) {
      String payload = http.getString();
      StaticJsonDocument<200> responseDoc;
      deserializeJson(responseDoc, payload);

      bool led1 = responseDoc["led1"] | false;
      bool led2 = responseDoc["led2"] | false;

      digitalWrite(LED1_PIN, led1);
      digitalWrite(LED2_PIN, led2);
    }
    http.end();
  }

  delay(200); // تحديث كل ثانية
}
