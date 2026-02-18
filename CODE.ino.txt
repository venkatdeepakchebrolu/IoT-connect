#include <ESP8266WiFi.h>
#include <Firebase_ESP_Client.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Wire.h>

// WiFi credentials
#define WIFI_SSID "SSID_HERE"
#define WIFI_PASSWORD "PASSWORD_HERE"

// Firebase configuration
#define FIREBASE_HOST "https://ping- pong-esp-default-rtdb.firebaseio.com/"
#define FIREBASE_AUTH "580f6e6e3fcadd46d"

// OLED configuration
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Firebase objects
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

String messageForOLED = "Waiting for data...";

void streamCallback(StreamData data) {
  Serial.print("Firebase Event: ");
  Serial.print(data.dataPath());
  Serial.print(" -> ");

  if (data.dataType() == "string") {
    messageForOLED = data.stringData();
    Serial.println("New message: " + messageForOLED);
  } else {
    Serial.println("Received non-string data or a different type of event.");
  }
}

void streamTimeoutCallback(bool timeout) {
  if (timeout) {
    Serial.println("Stream timeout, please check your network connection.");
  }
}

void setup() {
  Serial.begin(115200);
  delay(100);

  // Initialize OLED display
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for (;;);
  }

  display.display();
  delay(2000);
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("Connecting to WiFi...");
  display.display();

  // Connect to WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    display.print(".");
    display.display();
  }

  Serial.println("\nWiFi Connected!");
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("WiFi Connected!");
  display.display();
  delay(1000);

  // Configure Firebase
  config.database_url = FIREBASE_HOST;
  config.signer.tokens.legacy_token = FIREBASE_AUTH;

  // Initialize Firebase
  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);

  Serial.print("Listening for Firebase data at: ");
  Serial.println("/IOT_APP_interface");

  if (!Firebase.RTDB.beginStream(&fbdo, "/IOT_APP_interface")) {
    Serial.println("Failed to start Firebase stream.");
    Serial.println("Error: " + fbdo.errorReason());
  }

  Firebase.RTDB.setStreamCallback(&fbdo, streamCallback, streamTimeoutCallback);

  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("Firebase Ready!");
  display.setCursor(0, 10);
  display.println(messageForOLED);
  display.display();
}

void loop() {
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("From Firebase:");
  display.setCursor(0, 10);
  display.println(messageForOLED);
  display.display();

  delay(500);
}
