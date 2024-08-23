#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>

#define WIFI_SSID "USERNAME"
#define WIFI_PASSWORD "PASSWORD"

#define DHTPIN 26      // Define the DHT11 data pin
#define DHTTYPE DHT11 // Define the type of DHT sensor (DHT11)

DHT dht(DHTPIN, DHTTYPE);

const char *serverUrl = "https://console.thingzmate.com/api/v1/device-types/esp31/devices/esp31/uplink"; // Replace with your server endpoint
String AuthorizationToken = "Bearer 1d3df63191d1438ced97e1a0909fc623";

void setup() {
  Serial.begin(115200);
  delay(4000); // Delay to let serial settle

  // Connect to WiFi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }
  Serial.println("Connected to WiFi");

  // Initialize the DHT sensor
  dht.begin();
}

void loop() {
  float Temperature = dht.readTemperature();
  float Humidity = dht.readHumidity();
  int Raw_Value = 100; // Example value, replace with your sensor reading if needed

  // Generate a random pH value between 0 and 14
  float pH = random(0, 1400) / 100.0;

  // Check if any reads failed and exit early (to try again).
  if (isnan(Temperature) || isnan(Humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  HTTPClient http;
  http.begin(serverUrl);
  http.addHeader("Content-Type", "application/json");
  http.addHeader("Authorization", AuthorizationToken); // Authorization token
  
  // Create JSON payload
  String payload = "{\"temperature\":" + String(Temperature) + ",\"humidity\":" + String(Humidity) + ",\"Raw_Value\":" + String(Raw_Value) + ",\"pH\":" + String(pH) + "}";
  
  // Send POST request
  int httpResponseCode = http.POST(payload);

  if (httpResponseCode > 0) {
    String response = http.getString();
    Serial.println("HTTP Response code: " + String(httpResponseCode));
    Serial.println(response);
  } else {
    Serial.print("Error code: ");
    Serial.println(httpResponseCode);
  }

  http.end(); // Free resources
  delay(10000); // Wait for 10 seconds before sending next request
}
