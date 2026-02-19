Arduino Code

#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <RTClib.h>
#include <OneWire.h>
#include <DallasTemperature.h>

// Pins
const int chipSelect = 10;
const int lightPin = A0;
const int oneWireBus = 2;

// Setup OneWire and DallasTemperature
OneWire oneWire(oneWireBus);
DallasTemperature sensors(&oneWire);

// RTC
RTC_DS3231 rtc; // optional

// Sampling settings
const unsigned long sampleInterval = 1000; // 1 sample per second
unsigned long lastSampleTime = 0;

// SD file
File dataFile;
int sampleCount = 0; // counter for 100 samples

void setup() {
  Serial.begin(9600);

  // SD card init
  if (!SD.begin(chipSelect)) {
    Serial.println("SD init failed!");
    while (1);
  }
  Serial.println("SD initialized.");

  // DS18B20 init
  sensors.begin();

  // RTC init
  if (!rtc.begin()) {
    Serial.println("RTC not found!");
  }

  // Create CSV file
  dataFile = SD.open("data.csv", FILE_WRITE);
  if (dataFile) {
    dataFile.println("Timestamp,Temperature_C,Light");
    dataFile.close();
  }
}

void loop() {
  unsigned long currentTime = millis();

  if (currentTime - lastSampleTime >= sampleInterval && sampleCount < 100) {
    lastSampleTime = currentTime;
    sampleCount++;

    // Read sensors
    int lightVal = analogRead(lightPin);
    sensors.requestTemperatures();
    float temperatureC = sensors.getTempCByIndex(0);

    // Timestamp
    DateTime now = rtc.now();
    String timestamp = String(now.year()) + "-" + String(now.month()) + "-" + String(now.day()) + " "
                     + String(now.hour()) + ":" + String(now.minute()) + ":" + String(now.second());

    // Write to SD
    dataFile = SD.open("data.csv", FILE_WRITE);
    if (dataFile) {
      dataFile.print(timestamp);
      dataFile.print(",");
      dataFile.print(temperatureC);
      dataFile.print(",");
      dataFile.println(lightVal);
      dataFile.close();
      Serial.print("Sample ");
      Serial.print(sampleCount);
      Serial.println(" saved.");
      Serial.println(timestamp);
      Serial.println(temperatureC);
      Serial.println(lightVal);
    } else {
      Serial.println("Error opening file");
    }
  }

  // Optional: stop after 100 samples
  if (sampleCount >= 100) {
    Serial.println("Collected 100 samples. Stopping loop.");
    while (1); // stop
  }
}
