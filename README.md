#include <HTTPClient.h>
#include <WiFi.h>
#include <ArduinoJson.h>
#include <Wire.h>
#include <DHT.h>
#include <LiquidCrystal.h>
const int rs = 5, en = 18, d4 = 19, d5 = 21, d6 = 22, d7 = 23;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
const int lm35_pin = 36;
int temp;
const int Respiratorysensor = 35;
int count, count1;
int R, R1;
int HEART = 34;
int H, H1;
#define DHTPIN 26
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);
float hum;
float temp1, T, temp11;
String sensor1_status;
String sensor2_status;
String sensor3_status;
String sensor4_status;
String sensor5_status;
String sensor6_status;
String sensor7_status;
String sensor8_status;
String sms_status;

void setup() {
  lcd.begin(16, 2);
  dht.begin();
  Serial.begin(9600);
  pinMode(lm35_pin, INPUT);
  pinMode(Respiratorysensor, INPUT);
  pinMode(HEART, INPUT);
  WiFi.begin("iotbegin471", "iotbegin471");  //WiFi connection
  while (WiFi.status() != WL_CONNECTED) {
    lcd.setCursor(0, 0);
    lcd.print("Connecting to  ");
    lcd.setCursor(0, 1);
    lcd.print("     iotbegin471");
    Serial.println("Waiting for Wi-Fi connection");
  }

  lcd.setCursor(0, 0);
  lcd.print("    SOLDIER     ");
  lcd.setCursor(0, 1);
  lcd.print("MONITORINGSYSTEM");
  delay(3000);
  lcd.clear();
}

void loop() {
  temp = analogRead(lm35_pin);
  Serial.println(temp);
  if (temp < 400) {
    temp11 = 0.00;  //
  } else {
    temp11 = (temp / 8.18);
  }
  if ((temp11 < 95.0)&&(temp11 != 0.00)) {
    temp1 = 96;
  } else {
    temp1 = temp11;
  }
  if (temp1 <= 96) {
    temp1 = 0;
  }
  R = analogRead(Respiratorysensor);
  if (R == 4095) {
    R1 = 0;
  } else {
    count = count + 2;
    if (R < 15) {
      R1 = 0;
    } else if ((R > 0) && (R < 1000)) {
      R1 = 16;
    } else if ((R > 1000) && (R < 2500)) {
      R1 = 23;
    } else if ((R > 2500) && (R < 4000)) {
      R1 = 28;
    }
    if (R1 > 28) {
      count = 0;
    }
  }
  H = analogRead(HEART);

  if (H == 4095) {
    H1 = 0;
  } else {
    count = count + 2;
    if (H == 0) {
      H1 = 69 + count;
    } else if ((H > 0) && (H < 500)) {
      H1 = 78;
    } else if ((H > 500) && (H < 1000)) {
      H1 = 82;
    } else if ((H > 1000) && (H < 1500)) {
      H1 = 85;
    } else if ((H > 1500) && (H < 2500)) {
      H1 = 88;
    } else if ((H > 2500) && (H < 4000)) {
      H1 = 95;
    }
    if (H1 > 99) {
      count = 0;
    }
  }
  T = dht.readTemperature();
  hum = dht.readHumidity();

  lcd.setCursor(0, 1);
  lcd.print("H:");
  lcd.print(H1);
  lcd.print("   ");

  lcd.setCursor(0, 0);
  lcd.print("R:");
  lcd.print(R1);
  lcd.print("   ");

  lcd.setCursor(8, 0);
  lcd.print("T:");
  lcd.print(temp1);
  lcd.print("   ");

  lcd.setCursor(8, 1);
  lcd.print("ET:");
  lcd.print(T);
  lcd.print("   ");
  delay(1000);

  if (H1 > 85) {
    sensor1_status = "H:" + String(H1) + " HEARTBEAT ABNORMAL ";
  } else if (H1 == 0) {
    sensor1_status = "H:" + String(H1);
  } else {
    sensor1_status = "H:" + String(H1) + " HEARTBEAT NORMAL ";
  }

  if (R1 > 20) {
    sensor2_status = "R:" + String(R1) + " RESPIRATORY ABNORMAL ";
  } else if (R1 == 0) {
    sensor2_status = "R:" + String(R1);
  } else {
    sensor2_status = "R:" + String(R1) + " RESPIRATORY NORMAL ";
  }

  if (temp1 > 96) {
    sensor3_status = ":T" + String(temp1) + " TEMPERATURE ABNORMAL ";
  } else if (temp1 == 0) {
    sensor3_status = "T:" + String(temp1);
  } else {
    sensor3_status = "T:" + String(temp1) + "TEMPERATURE NORMAL ";
  }
  sensor4_status = "ET:" + String(T);
  sensor5_status = "EH:" + String(hum);
  iot();
}

void iot() {
  if (WiFi.status() == WL_CONNECTED) {
    DynamicJsonDocument jsonBuffer(JSON_OBJECT_SIZE(3) + 300);
    JsonObject root = jsonBuffer.to<JsonObject>();

    root["sensor1"] = sensor1_status;
    root["sensor2"] = sensor2_status;
    root["sensor3"] = sensor3_status;
    root["sensor4"] = sensor4_status;
    root["sensor5"] = sensor5_status;
    root["sensor6"] = sensor6_status;
    root["sensor7"] = sensor7_status;
    root["sensor8"] = sensor8_status;
    root["sms"] = sms_status;
    String json;
    serializeJson(jsonBuffer, json);
    if (sensor1_status != "null") {
      HTTPClient http;                                   //Declare object of class HTTPClient
      http.begin("http://iotbegineer.com/api/sensors");  //Specify request destination
      http.addHeader("username", "iotbegin471");         //Specify content-type header
      http.addHeader("Content-Type", "application/json");
      int httpCode = http.POST(json);     //Send the request
      String payload = http.getString();  //Get the response payload
      http.end();
    }
  } else {
    Serial.println("Error in WiFi connection");
  }
}
