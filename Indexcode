#include <LiquidCrystal_I2C.h>
#include <WiFiClientSecure.h>
#define BLYNK_PRINT Serial
#define BLYNK_TEMPLATE_ID "TMPL3yAKF8WH8"
#define BLYNK_TEMPLATE_NAME "Air and water quality index"
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <Wire.h>
#include <DHT.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);
#define LDR_SENSOR_PIN D0
#define LED_PIN D7 // Change to the GPIO pin connected to the LED
DHT dht(D3, DHT11); //(sensor pin, sensor type)

char auth[] = "-H-Kg8gBGIbV19VdBTdLZUQthWKJzctV"; // Enter your Auth token
char ssid[] = "Sahil"; // Enter your WIFI name
char pass[] = "Sahil@03"; // Enter your WIFI password

// Google Sheets parameters
const char* host = "script.google.com";
const int httpsPort = 443;
WiFiClientSecure client;
String GAS_ID = "AKfycbzYAmM52p3gIOCMngDrCbGJl6UtW1OoJfWbab7Wlvab5JWkpu-aKHiv8SwA509DGz1E"; // Google Apps Script ID

BlynkTimer timer;

// Variables to store the latest readings
float latestTemp = 0;
float latestHum = 0;
int latestRain = 0;
String latestLedStatus = "OFF";  // Change to String to store "ON" or "OFF"

// Function to read DHT sensor and update the latest readings
void readDHTSensor() {
  latestHum = dht.readHumidity();
  latestTemp = dht.readTemperature();

  if (isnan(latestHum) || isnan(latestTemp)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  lcd.setCursor(0, 0);
  lcd.print("Temp : ");
  lcd.print(latestTemp);
  lcd.setCursor(0, 1);
  lcd.print("Humi : ");
  lcd.print(latestHum);

  Blynk.virtualWrite(V0, latestTemp);
  Blynk.virtualWrite(V1, latestHum);
}

// Function to read the rain sensor and update the latest reading
void readMoisture() {
  latestRain = analogRead(A0);
  latestRain = map(latestRain, 0, 1023, 0, 100);
  Blynk.virtualWrite(V2, latestRain);

  Serial.print("Rain Percentage: ");
  Serial.println(latestRain);
}

// Function to read LDR sensor and update the latest LED status
void readLDRSensor() {
  int value = digitalRead(LDR_SENSOR_PIN); // Read the digital value from the LDR pin
  int mappedValue = map(value, LOW, HIGH, 0, 1); // Map the digital value to the range 0-1
  Serial.print("LDR Mapped Value: ");
  Serial.println(mappedValue);

  Blynk.virtualWrite(V4, mappedValue); // Send the LDR sensor value to Blynk app

  // Update latestLedStatus based on mapped value
  if (mappedValue == 1) {
    latestLedStatus = "ON";
    digitalWrite(LED_PIN, HIGH); // Turn on the LED when it's dark
  } else {
    latestLedStatus = "OFF";
    digitalWrite(LED_PIN, LOW); // Turn off the LED when it's bright
  }

  Serial.print("LED Status: ");
  Serial.println(latestLedStatus);
}

// Function to send all data to Google Sheets
void sendDataToGoogle() {
  if (!client.connect(host, httpsPort)) {
    Serial.println("Connection to Google host failed!");
    return;
  }

  // Prepare the URL with all values
  String url = "/macros/s/" + GAS_ID + "/exec?";
  url += "temperature=" + String(latestTemp) + "&";
  url += "humidity=" + String(latestHum) + "&";
  url += "rainfall=" + String(latestRain) + "&";
  url += "led=" + latestLedStatus;  // Sending "ON" or "OFF" as a string

  Serial.print("Requesting URL: ");
  Serial.println(url);

  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" +
               "Connection: close\r\n\r\n");

  while (client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      break;
    }
  }
  
  String line = client.readStringUntil('\n');
  Serial.print("Reply: ");
  Serial.println(line);
}

void setup() {
  Serial.begin(9600);
  Wire.begin(D2, D1);
  lcd.begin();
  lcd.backlight();
  Serial.println("LCD initialized");

  Blynk.begin(auth, ssid, pass);
  dht.begin();

  pinMode(LED_PIN, OUTPUT); // Set LED pin as output
  digitalWrite(LED_PIN, LOW); // Initially turn off the LED
  pinMode(LDR_SENSOR_PIN, INPUT);

  client.setInsecure(); // For HTTPS connection without certificates

  // Setup the timers for each sensor read
  timer.setInterval(2000L, readDHTSensor);
  timer.setInterval(2000L, readMoisture);
  timer.setInterval(2000L, readLDRSensor);

  // Send data to Google Sheets every 5 seconds (you can adjust this interval)
  timer.setInterval(5000L, sendDataToGoogle);
}

void loop() {
  Blynk.run();
  timer.run();
}
