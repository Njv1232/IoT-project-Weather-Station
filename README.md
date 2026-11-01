# IOT PROJECT
## ESP32 Weather Station with Heat Danger Warning

An ESP32-based weather station that measures **temperature**, **humidity** and **rainfall**, shows the readings on an **OLED display**, and serves a live **web dashboard** from its own WiFi network. When temperature and humidity together reach dangerous levels, it shows a **heat warning** on both the OLED and the web page.

---

## Features

- Reads temperature and humidity with a **DHT11** sensor
- Reads rainfall with a **DFRobot I2C Rainfall Sensor** (tipping bucket)
- Shows live data on a **1.3" SH1106 OLED** (128x64, I2C)
- **Heat danger warning** with 4 levels (Caution, Warning, Danger, Extreme), based on temperature **and** humidity
- ESP32 creates its **own WiFi network** (no router needed)
- **Captive portal**: phones open the dashboard automatically after joining
- Web dashboard refreshes every 2 seconds, with a colour-coded warning banner
- **JSON API** at `/data` with all the readings
- Rain sensor is optional: the program keeps working if it is not connected

---

## Hardware

| Component | Notes |
|---|---|
| ESP32 Dev Module | Any ESP32 dev board |
| DHT11 temperature/humidity sensor | 3-pin module or 4-pin with 10k resistor |
| 1.3" OLED, 128x64, I2C | SH1106 controller, address `0x3C` |
| DFRobot I2C Rainfall Sensor | Optional |
| Breadboard and jumper wires | |
| USB cable (data capable) | For programming and power |

### Wiring

| Part | Pin | ESP32 pin |
|---|---|---|
| DHT11 | VCC (+) | 3.3V |
| DHT11 | GND (-) | GND |
| DHT11 | DATA (S/OUT) | GPIO 4 |
| OLED | VCC | 3.3V |
| OLED | GND | GND |
| OLED | SDA | GPIO 21 |
| OLED | SCK / SCL | GPIO 22 |
| Rain sensor | SDA | GPIO 21 (shared with OLED) |
| Rain sensor | SCL | GPIO 22 (shared with OLED) |
| Rain sensor | VCC / GND | Check the sensor label (3.3V or 5V) / GND |

The OLED and rain sensor share the same I2C wires because they use different addresses.

---

## Software Setup

### 1. Arduino IDE and board package

1. Install the **Arduino IDE 2.x**.
2. Open **Boards Manager**, search for `esp32` by **Espressif Systems** and install it (tested with version 3.3.12).
3. Select **ESP32 Dev Module** as the board.
4. If the COM port does not appear, install the **CP210x** or **CH340** USB driver (depending on the chip on your board) and use a data-capable USB cable.

### 2. Libraries

Install these from **Sketch > Include Library > Manage Libraries**:

- `Adafruit SH110X`
- `Adafruit GFX Library`
- `DHT sensor library` (Adafruit)
- `Adafruit Unified Sensor`
- `DFRobot_RainfallSensor` (if not in Library Manager, download the ZIP from DFRobot's GitHub and use **Add .ZIP Library**)

`WiFi`, `WebServer` and `DNSServer` come with the ESP32 board package.

### 3. Upload

1. Open `esp32_weather_alert/esp32_weather_alert.ino`.
2. Select the correct board and COM port.
3. Click **Upload**. If it stays on "Connecting...", hold the **BOOT** button until uploading starts.

---

## Using the Station

1. Power the ESP32.
2. On a phone or laptop, join the WiFi network:
   - **Name:** `ESP32-Weather`
   - **Password:** `12345678`
3. A "Sign in to network" notification should open the dashboard automatically. If not, open a browser and go to **http://192.168.4.1** (use `http`, not `https`).

The same address is shown at the bottom of the OLED, with the number of connected devices in brackets.

To change the WiFi name or password, edit `AP_SSID` and `AP_PASS` at the top of the sketch (password must be at least 8 characters).

---

## Heat Warning Levels

A level is reached only when **both** the temperature **and** the humidity are at or above that level's limits. The highest matching level is shown.

| Level | Temperature | Humidity | Message |
|---|---|---|---|
| NORMAL | below limits | below limits | Conditions normal |
| CAUTION | 28 C or more | 60 % or more | Limit outdoor activity |
| WARNING | 32 C or more | 60 % or more | Drink water, take rest |
| DANGER | 35 C or more | 55 % or more | Heat stroke risk! |
| EXTREME | 40 C or more | 50 % or more | LIFE THREATENING! (blinks) |

The limits are defined by the `T_` and `H_` constants near the top of the sketch and can be changed.

> **Disclaimer:** The DHT11 is accurate to only about +/-2 C and +/-5 % humidity. These warnings are a guide and must not be used as the only basis for safety decisions. Follow official weather and health alerts.

---

## JSON API

`GET http://192.168.4.1/data` returns:

```json
{
  "humidity": 65,
  "temperature": 31,
  "heat_index": 36,
  "level": 2,
  "level_text": "WARNING",
  "advice": "Drink water, take rest",
  "rain_total": 0.00,
  "rain_1h": 0.00,
  "tips": 0,
  "rain_ok": true
}
```

| Field | Meaning |
|---|---|
| `humidity` | Relative humidity in % (`null` if the sensor read failed) |
| `temperature` | Temperature in degrees C (`null` if the sensor read failed) |
| `heat_index` | "Feels like" temperature in degrees C |
| `level` | Warning level, 0 (normal) to 4 (extreme) |
| `level_text` / `advice` | Text for the current level |
| `rain_total` | Accumulated rainfall in mm |
| `rain_1h` | Rainfall in the last hour in mm |
| `tips` | Raw tipping-bucket count |
| `rain_ok` | `false` if the rain sensor is not connected |

---

## Project Structure

```
esp32-weather-station/
├── README.md
├── esp32_weather_alert/
│   └── esp32_weather_alert.ino     # main sketch (use this one)
├── docs/
│   ├── wiring.jpg                  # photo or diagram of your wiring
│   └── dashboard.png               # screenshot of the web page
└── LICENSE
```

---

## Troubleshooting

| Problem | What to try |
|---|---|
| Board not detected in Arduino IDE | Use a data USB cable, install the CP210x/CH340 driver, pick the COM port |
| OLED shows noise or garbage | The display uses SH1106, so use `Adafruit_SH110X` (not SSD1306) |
| OLED blank, "OLED not found" in Serial Monitor | Check SDA/SCL wires, try address `0x3D` |
| Page says "refused to connect" | Use `http://192.168.4.1`, not `https://` |
| Page does not open on a phone | Turn off mobile data and VPN, tap "Stay connected" if asked, forget and rejoin the network |
| "Rain sensor: not connected" | Check SDA/SCL wires and the sensor's power |
| Humidity stuck at the same value | See the known issue below |

---

## Known Issue

During testing, one DHT11 sensor showed the humidity stuck at exactly **98.0 %** while the temperature changed normally. This is a failed humidity element in the sensor, not a code or calibration problem. Replacing the sensor fixes it. Good replacements:

- **DHT22 (AM2302):** same wiring, change `#define DHTTYPE DHT11` to `DHT22`
- **BME280 / AHT20:** I2C sensors, more reliable (requires small code changes)

---

## Possible Improvements

- Detect a stuck humidity sensor and show "Humidity sensor fault"
- Send data to an MQTT broker or a cloud dashboard
- Log readings to an SD card
- Buzzer or LED alarm for the highest warning level

---

## Full Source Code

The complete sketch (`esp32_weather_alert/esp32_weather_alert.ino`) is included below for reference.

<details>
<summary>Click to show the code</summary>

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <DNSServer.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SH110X.h>
#include <DHT.h>
#include "DFRobot_RainfallSensor.h"

// ---- Access point settings (the ESP32 creates this WiFi network) ----
const char* AP_SSID = "ESP32-Weather";
const char* AP_PASS = "12345678";   // must be at least 8 characters

// ---- Pin settings ----
#define DHTPIN   4
#define DHTTYPE  DHT11
#define SDA_PIN  21
#define SCL_PIN  22

// ---- OLED settings (SH1106, 1.3" 128x64) ----
#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 64
#define OLED_ADDR     0x3C

// ---- Warning thresholds: BOTH temperature AND humidity must reach the limit ----
// Temperature in degrees C, humidity in %. Change them as you like.
#define T_CAUTION   28.0
#define H_CAUTION   60.0

#define T_WARNING   32.0
#define H_WARNING   60.0

#define T_DANGER    35.0
#define H_DANGER    55.0

#define T_EXTREME   40.0
#define H_EXTREME   50.0

DHT dht(DHTPIN, DHTTYPE);
Adafruit_SH1106G display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
DFRobot_RainfallSensor_I2C rainSensor(&Wire);
WebServer server(80);
DNSServer dnsServer;

bool rainSensorOk = false;
float humidity = NAN;
float temperature = NAN;
float heatIndex = NAN;
int warnLevel = 0;
float rainTotal = 0;
float rain1h = 0;
uint32_t rainRaw = 0;
unsigned long lastRead = 0;
bool blinkState = false;

const char* LEVEL_TEXT[5]  = {"NORMAL", "CAUTION", "WARNING", "DANGER", "EXTREME"};
const char* LEVEL_ADVICE[5] = {
  "Conditions normal",
  "Limit outdoor activity",
  "Drink water, take rest",
  "Heat stroke risk!",
  "LIFE THREATENING!"
};

// ---- Web page ----
const char INDEX_HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>ESP32 Weather Station</title>
<style>
  body { font-family: Arial, sans-serif; background: #0f172a; color: #e2e8f0;
         margin: 0; padding: 20px; text-align: center; }
  h1 { font-size: 1.5rem; margin-bottom: 16px; }
  #banner { max-width: 700px; margin: 0 auto 16px auto; padding: 14px; border-radius: 12px;
            font-weight: bold; font-size: 1.2rem; background: #166534; color: #fff; }
  #banner small { display: block; font-weight: normal; font-size: 0.9rem; margin-top: 4px; }
  .l0 { background: #166534 !important; }
  .l1 { background: #a16207 !important; }
  .l2 { background: #c2410c !important; }
  .l3 { background: #b91c1c !important; }
  .l4 { background: #7f1d1d !important; animation: blink 1s infinite; }
  @keyframes blink { 50% { opacity: 0.45; } }
  .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
          gap: 14px; max-width: 700px; margin: 0 auto; }
  .card { background: #1e293b; border-radius: 12px; padding: 18px; }
  .label { font-size: 0.85rem; color: #94a3b8; }
  .value { font-size: 2rem; font-weight: bold; margin-top: 6px; }
  .unit { font-size: 1rem; color: #94a3b8; }
  #status { margin-top: 20px; font-size: 0.8rem; color: #64748b; }
</style>
</head>
<body>
  <h1>ESP32 Weather Station</h1>
  <div id="banner" class="l0">Waiting for data...<small></small></div>
  <div class="grid">
    <div class="card"><div class="label">Humidity</div>
      <div class="value"><span id="hum">--</span><span class="unit"> %</span></div></div>
    <div class="card"><div class="label">Temperature</div>
      <div class="value"><span id="temp">--</span><span class="unit"> &deg;C</span></div></div>
    <div class="card"><div class="label">Heat index (feels like)</div>
      <div class="value"><span id="hi">--</span><span class="unit"> &deg;C</span></div></div>
    <div class="card"><div class="label">Rain total</div>
      <div class="value"><span id="rain">--</span><span class="unit"> mm</span></div></div>
    <div class="card"><div class="label">Rain (last hour)</div>
      <div class="value"><span id="rain1h">--</span><span class="unit"> mm</span></div></div>
    <div class="card"><div class="label">Bucket tips</div>
      <div class="value"><span id="tips">--</span></div></div>
  </div>
  <div id="status">Connecting...</div>
<script>
  function show(id, v, d) {
    document.getElementById(id).textContent = (v === null) ? "--" : Number(v).toFixed(d);
  }
  async function update() {
    try {
      const r = await fetch("/data", { cache: "no-store" });
      const d = await r.json();
      show("hum", d.humidity, 0);
      show("temp", d.temperature, 0);
      show("hi", d.heat_index, 0);
      show("rain", d.rain_total, 2);
      show("rain1h", d.rain_1h, 2);
      show("tips", d.tips, 0);
      const b = document.getElementById("banner");
      b.className = "l" + d.level;
      b.innerHTML = (d.level > 0 ? "&#9888; " : "") + d.level_text +
                    "<small>" + d.advice + "</small>";
      document.getElementById("status").textContent =
        "Updated " + new Date().toLocaleTimeString() +
        (d.rain_ok ? "" : " | Rain sensor not connected");
    } catch (e) {
      document.getElementById("status").textContent = "Connection lost...";
    }
  }
  update();
  setInterval(update, 2000);
</script>
</body>
</html>
)rawliteral";

String num(float v, int decimals) {
  return isnan(v) ? String("null") : String(v, decimals);
}

void handleRoot() {
  server.sendHeader("Cache-Control", "no-cache");
  server.send_P(200, "text/html", INDEX_HTML);
}

void handleData() {
  String json = "{";
  json += "\"humidity\":" + num(humidity, 0) + ",";
  json += "\"temperature\":" + num(temperature, 0) + ",";
  json += "\"heat_index\":" + num(heatIndex, 0) + ",";
  json += "\"level\":" + String(warnLevel) + ",";
  json += "\"level_text\":\"" + String(LEVEL_TEXT[warnLevel]) + "\",";
  json += "\"advice\":\"" + String(LEVEL_ADVICE[warnLevel]) + "\",";
  json += "\"rain_total\":" + num(rainTotal, 2) + ",";
  json += "\"rain_1h\":" + num(rain1h, 2) + ",";
  json += "\"tips\":" + String((unsigned long)rainRaw) + ",";
  json += "\"rain_ok\":" + String(rainSensorOk ? "true" : "false");
  json += "}";
  server.sendHeader("Cache-Control", "no-store");
  server.send(200, "application/json", json);
}

// Captive portal: any unknown address is sent to the main page,
// so phones open the page automatically after joining the WiFi.
void handleNotFound() {
  String url = "http://" + WiFi.softAPIP().toString() + "/";
  server.sendHeader("Location", url, true);
  server.send(302, "text/plain", "");
}

// A level is reached only when temperature AND humidity are both
// at or above that level's limits. The highest matching level wins.
int calcWarnLevel(float t, float h) {
  if (isnan(t) || isnan(h)) return 0;
  if (t >= T_EXTREME && h >= H_EXTREME) return 4;
  if (t >= T_DANGER  && h >= H_DANGER)  return 3;
  if (t >= T_WARNING && h >= H_WARNING) return 2;
  if (t >= T_CAUTION && h >= H_CAUTION) return 1;
  return 0;
}

void readSensors() {
  humidity = dht.readHumidity();
  temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    heatIndex = NAN;
    warnLevel = 0;
    Serial.println("DHT11 read failed");
  } else {
    heatIndex = dht.computeHeatIndex(temperature, humidity, false);  // Celsius
    warnLevel = calcWarnLevel(temperature, humidity);
    Serial.printf("Humidity: %.0f %%  Temp: %.0f C  Heat index: %.0f C  Level: %s\n",
                  humidity, temperature, heatIndex, LEVEL_TEXT[warnLevel]);
  }

  if (!rainSensorOk) {
    rainSensorOk = rainSensor.begin();
  }
  if (rainSensorOk) {
    rainTotal = rainSensor.getRainfall();
    rain1h = rainSensor.getRainfall(1);
    rainRaw = rainSensor.getRawData();
    Serial.printf("Rain total: %.2f mm  1h: %.2f mm  Tips: %lu\n",
                  rainTotal, rain1h, (unsigned long)rainRaw);
  } else {
    Serial.println("Rain sensor not connected");
  }
}

void updateDisplay() {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SH110X_WHITE);

  // Line 1: humidity
  display.setCursor(0, 0);
  if (isnan(humidity)) {
    display.print("Humidity: error");
  } else {
    display.print("Humidity: ");
    display.print(humidity, 0);
    display.print(" %");
  }

  // Line 2: temperature + heat index
  display.setCursor(0, 9);
  if (isnan(temperature)) {
    display.print("Temp: error");
  } else {
    display.print("Temp:");
    display.print(temperature, 0);
    display.print("C HeatIdx:");
    display.print(heatIndex, 0);
    display.print("C");
  }

  // Line 3: rain
  display.setCursor(0, 18);
  if (rainSensorOk) {
    display.print("R:");
    display.print(rainTotal, 2);
    display.print(" 1h:");
    display.print(rain1h, 2);
    display.print(" T:");
    display.print(rainRaw);
  } else {
    display.print("Rain sensor: none");
  }

  // Line 4: advice text
  display.setCursor(0, 28);
  display.print(LEVEL_ADVICE[warnLevel]);

  // Warning banner
  if (warnLevel > 0) {
    // Level 4 blinks (inverts) on every update
    bool filled = (warnLevel < 4) || blinkState;
    if (filled) {
      display.fillRect(0, 38, SCREEN_WIDTH, 16, SH110X_WHITE);
      display.setTextColor(SH110X_BLACK);
    } else {
      display.drawRect(0, 38, SCREEN_WIDTH, 16, SH110X_WHITE);
      display.setTextColor(SH110X_WHITE);
    }
    display.setTextSize(2);
    int textWidth = strlen(LEVEL_TEXT[warnLevel]) * 12;
    display.setCursor((SCREEN_WIDTH - textWidth) / 2, 40);
    display.print(LEVEL_TEXT[warnLevel]);
    display.setTextSize(1);
    display.setTextColor(SH110X_WHITE);
  } else {
    display.setCursor(0, 42);
    display.print("Status: NORMAL");
  }

  // Bottom line: IP address + number of connected devices
  display.setCursor(0, 56);
  display.print(WiFi.softAPIP());
  display.print(" (");
  display.print(WiFi.softAPgetStationNum());
  display.print(")");

  display.display();
  blinkState = !blinkState;
}

void setup() {
  Serial.begin(115200);
  Wire.begin(SDA_PIN, SCL_PIN);
  dht.begin();

  if (!display.begin(OLED_ADDR, true)) {
    Serial.println("OLED not found. Check wiring/address.");
    while (true) delay(10);
  }
  display.clearDisplay();
  display.setTextColor(SH110X_WHITE);
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println("Starting WiFi...");
  display.display();

  // Rain sensor (don't block forever if missing)
  for (int i = 0; i < 5 && !rainSensorOk; i++) {
    rainSensorOk = rainSensor.begin();
    if (!rainSensorOk) delay(300);
  }

  // Start the ESP32's own WiFi network (channel 1, visible, max 4 devices)
  WiFi.mode(WIFI_AP);
  bool apOk = WiFi.softAP(AP_SSID, AP_PASS, 1, 0, 4);
  WiFi.setSleep(false);   // keeps the connection stable for phones
  delay(500);

  if (apOk) {
    Serial.print("Access point started. WiFi name: ");
    Serial.println(AP_SSID);
    Serial.print("Open this address in your browser: http://");
    Serial.println(WiFi.softAPIP());
  } else {
    Serial.println("Failed to start access point.");
  }

  // DNS server: answers every name with the ESP32's address (captive portal)
  dnsServer.start(53, "*", WiFi.softAPIP());

  server.on("/", handleRoot);
  server.on("/data", handleData);
  server.onNotFound(handleNotFound);
  server.begin();

  readSensors();
  updateDisplay();
  lastRead = millis();
}

void loop() {
  dnsServer.processNextRequest();
  server.handleClient();

  if (millis() - lastRead >= 2000) {
    lastRead = millis();
    readSensors();
    updateDisplay();
  }
}
```

</details>

---

## License

Add your preferred license here (for example MIT) and include a `LICENSE` file in the repository.

## Authors

Add your name and team members here.
