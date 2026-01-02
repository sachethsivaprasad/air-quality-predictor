#include <DHT.h>

// Sensor Pin Definitions
#define MQ7_PIN A0        // Analog pin for MQ-7 (CO)
#define MQ135_PIN A1      // Analog pin for MQ-135 (Air Quality)
#define DHTPIN 4         // Digital pin for DHT11
#define DHTTYPE DHT11     // Type of DHT sensor

// DHT11 Sensor Initialization
DHT dht(DHTPIN, DHTTYPE);

// Calibration Constants (Update after calibration)
float R0_MQ7 = 13.53;     // Must be calibrated for MQ-7
float R0_MQ135 = 9.69;   // Must be calibrated for MQ-135

// Constants for Gas Curve (Datasheet)
float A_MQ7 = 99.042;    // A constant for MQ-7 CO
float B_MQ7 = -1.518;    // B constant for MQ-7 CO

float A_MQ135 = 116.602; // A constant for MQ-135
float B_MQ135 = -2.769;  // B constant for MQ-135

// Function to calculate resistance Rs from analog value
float getSensorResistance(int sensorPin) {
  int sensorValue = analogRead(sensorPin);
  float sensorVoltage = (sensorValue / 1023.0) * 5.0; // Convert to voltage
  float sensorResistance = ((5.0 * 10.0) / sensorVoltage) - 10.0; // Rs calculation with RL = 10K
  return sensorResistance;
}

// Function to calculate PPM using Rs/R0 ratio
float getPPM(float Rs, float R0, float A, float B) {
  float ratio = Rs / R0; // Rs/R0 ratio
  return A * pow(ratio, B); // Apply equation from datasheet
}

void setup() {
  Serial.begin(9600);
  dht.begin();
}

void loop() {
  // Read resistance values
  float Rs_MQ7 = getSensorResistance(MQ7_PIN);
  float Rs_MQ135 = getSensorResistance(MQ135_PIN);

  // Convert to PPM
  float ppm_CO = getPPM(Rs_MQ7, R0_MQ7, A_MQ7, B_MQ7);
  float ppm_AirQuality = getPPM(Rs_MQ135, R0_MQ135, A_MQ135, B_MQ135);

  // Read Temperature & Humidity
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // Print Data
  Serial.print("CO: ");
  Serial.print(ppm_CO);
  Serial.print(" | AirQualityPPM: ");
  Serial.print(ppm_AirQuality);
  Serial.print(" | Temperature: ");
  Serial.print(temperature);
  Serial.print("| Humidity: ");
  Serial.print(humidity);
  Serial.print("\n");
  delay(1000);
}
