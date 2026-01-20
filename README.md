#include <WiFi.h>
#include <WebServer.h>

// Replace with your WiFi credentials
const char* ssid = "123456789";
const char* password = "123456789";

// Define sensor input pins
const int PH_PIN = 34;
const int TURBIDITY_PIN = 35;
const int TDS_PIN = 32;

// Create web server on port 80
WebServer server(80);

float readPH() {
  int adcValue = analogRead(PH_PIN);
  float voltage = adcValue * (3.3 / 4095.0);
  float phValue = 3.5 * voltage;  // Example formula (adjust with calibration)
  return phValue;
}

float readTurbidity() {
  int adcValue = analogRead(TURBIDITY_PIN);
  float voltage = adcValue * (3.3 / 4095.0);
  float turbidity = 100 - (voltage * 100 / 3.3); // Simple linear example
  return turbidity;
}

float readTDS() {
  int adcValue = analogRead(TDS_PIN);
  float voltage = adcValue * (3.3 / 4095.0);
  float tdsValue = (133.42 * voltage * voltage * voltage - 255.86 * voltage * voltage + 857.39 * voltage) * 0.5; // Example
  return tdsValue;
}

// Serve the web page
void handleRoot() {
  float ph = readPH();
  float turbidity = readTurbidity();
  float tds = readTDS();

  String html = "<!DOCTYPE html><html><head><meta charset='UTF-8'><title>Water Quality</title>";
  html += "<style>body{font-family:sans-serif; text-align:center;} h1{color:#2E8B57;}</style></head><body>";
  html += "<h1>Water Quality Monitor</h1>";
  html += "<p><strong>pH:</strong> " + String(ph, 2) + "</p>";
  html += "<p><strong>Turbidity:</strong> " + String(turbidity, 2) + " %</p>";
  html += "<p><strong>TDS:</strong> " + String(tds, 2) + " ppm</p>";
  html += "</body></html>";

  server.send(200, "text/html", html);
}

void setup() {
  Serial.begin(115200);

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi..");
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("\nConnected to WiFi!");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  // Setup web server
  server.on("/", handleRoot);
  server.begin();
  Serial.println("HTTP server started");
}

void loop() {
  server.handleClient();
}
