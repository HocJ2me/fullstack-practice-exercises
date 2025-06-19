
# 📘 Chuỗi Bài Tập IoT cơ bản với ESP32 và ThingSpeak

## ✅ Bài 1: Kết nối WiFi với ESP32

**Mục tiêu:** Kết nối ESP32 với mạng WiFi và hiển thị trạng thái kết nối qua Serial Monitor.

```cpp
#include <WiFi.h>

const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.print("Đang kết nối");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nĐã kết nối!");
  Serial.print("Địa chỉ IP: ");
  Serial.println(WiFi.localIP());
}

void loop() {
}
```

---

## ✅ Bài 2: Gửi dữ liệu ngẫu nhiên lên ThingSpeak

**Mục tiêu:** Gửi giá trị nhiệt độ ngẫu nhiên lên ThingSpeak mỗi 15 giây.

```cpp
#include <WiFi.h>
#include "ThingSpeak.h"

const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";
WiFiClient client;

unsigned long channelID = 1234567;
const char* writeAPIKey = "YOUR_WRITE_API_KEY";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
  ThingSpeak.begin(client);
}

void loop() {
  float temp = random(200, 300) / 10.0;
  int status = ThingSpeak.writeField(channelID, 1, temp, writeAPIKey);
  if (status == 200) {
    Serial.println("Gửi thành công: " + String(temp));
  } else {
    Serial.println("Lỗi gửi dữ liệu: " + String(status));
  }
  delay(15000);
}
```

---

## ✅ Bài 3: Tạo Web Server hiển thị nhiệt độ

**Mục tiêu:** Tạo server web đơn giản hiển thị dữ liệu cảm biến (giả lập).

```cpp
#include <WiFi.h>

const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

WiFiServer server(80);

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  while(WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
  Serial.print("Địa chỉ IP: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    Serial.println("Client connected");
    String request = client.readStringUntil('\r');
    client.flush();

    float temp = random(200, 300) / 10.0;

    client.println("HTTP/1.1 200 OK");
    client.println("Content-Type: text/html");
    client.println();
    client.println("<html><body><h1>Nhiệt độ: " + String(temp) + " °C</h1></body></html>");
    delay(1);
    client.stop();
  }
}
```

---

## 🔁 Gợi ý bài tập tiếp theo

1. Đọc cảm biến thực (DHT11, LM35...) và gửi lên ThingSpeak.
2. Kết hợp Web Server + Cảm biến thực để hiển thị dữ liệu.
3. Gửi nhiều `field` cùng lúc lên ThingSpeak.
4. Tự động gửi dữ liệu theo điều kiện (nhiệt độ > 30 thì gửi cảnh báo).
5. Xử lý khi mất WiFi hoặc gửi thất bại.

---

**💡 Ghi chú:** Đừng quên đổi `Your_SSID`, `Your_PASSWORD`, và `YOUR_WRITE_API_KEY` thành thông tin thật của bạn.
