# Gps-tracking-device-plus-geofancing-project
Gps tracking device plus geofancing project
#include <WiFi.h>              // ESP32 WiFi Library
#include <TinyGPSPlus.h>       // GPS Library
#include <HTTPClient.h>        // HTTP Client for API Requests
#include <HardwareSerial.h>    // UART Communication

// GPS Setup
TinyGPSPlus gps;
HardwareSerial gpsSerial(2);   // UART2 for GPS (GPIO 16 = RX, GPIO 17 = TX)

// WiFi Credentials
const char* ssid = "Awais Ashraf 🙂";     // Your WiFi SSID (name)
const char* password = "Mani@123";        // Your WiFi Password

// Twilio Credentials
const String account_sid = "AC00505100e349f630cd58665a7b408c2c";  // Your Account SID
const String auth_token = "4b74a0ac911459dac186b6d8013a17e4";      // Your Auth Token
const String from_number = "+17755874430";                         // Your Twilio phone number
const String to_number = "+923080087463";                          // Your recipient phone number

void setup() {
  Serial.begin(9600);                  // Serial Monitor
  gpsSerial.begin(9600, SERIAL_8N1, 16, 17);  // GPS Module (TX on GPIO 16, RX on GPIO 17)

  // Connect to WiFi (STA Mode)
  WiFi.begin(ssid, password);
  Serial.println("Connecting to WiFi...");
  
  // Wait for connection
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.print(".");
  }

  Serial.println("Connected to WiFi");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());  // Display ESP32 IP Address
}

void loop() {
  // Read GPS data
  while (gpsSerial.available() > 0) {
    gps.encode(gpsSerial.read());
  }

  if (gps.location.isValid()) {
    double latitude = gps.location.lat();
    double longitude = gps.location.lng();
    double speed = gps.speed.kmph();

    // Create Google Maps link
    String googleMapsLink = "https://www.google.com/maps?q=" + String(latitude, 6) + "," + String(longitude, 6);

    // Prepare SMS message
    String message = "Current Location:\n";
    message += "Latitude: " + String(latitude, 6) + "\n";
    message += "Longitude: " + String(longitude, 6) + "\n";
    message += "Speed: " + String(speed) + " km/h\n";
    message += "Google Maps: " + googleMapsLink;

    sendSMS(message);  // Send the SMS
    delay(5000);      // Wait 1 minute before sending the next SMS
  } else {
    Serial.println("Waiting for GPS signal...");
  }
}

void sendSMS(String message) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String url = "https://api.twilio.com/2010-04-01/Accounts/" + account_sid + "/Messages.json";  // Correct endpoint

    http.begin(url);
    http.setAuthorization(account_sid.c_str(), auth_token.c_str());
    http.addHeader("Content-Type", "application/x-www-form-urlencoded");

    String postData = "From=" + from_number + "&To=" + to_number + "&Body=" + message;
    
    // Send POST request to Twilio
    int httpResponseCode = http.POST(postData);

    if (httpResponseCode == 201) {
      Serial.println("Message sent successfully!");
    } else {
      Serial.print("Error in sending message: ");
      Serial.println(httpResponseCode);
      Serial.println(http.getString());  // Print error response for debugging
    }

    http.end();
  } else {
    Serial.println("WiFi not connected");
  }
}


