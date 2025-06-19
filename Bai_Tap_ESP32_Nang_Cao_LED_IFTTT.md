
# 🔧 Bài Tập ESP32 Nâng Cao: Điều khiển và Cảnh Báo

---

## ✅ Bài 1: Web Server điều khiển LED qua trình duyệt

**Mục tiêu:** Bật/Tắt LED thông qua giao diện web.

**Phần cứng:**
- ESP32
- LED nối chân D2 (GPIO 2)

**Code:**

```cpp
#include <WiFi.h>

const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

WiFiServer server(80);
const int ledPin = 2;

void setup() {
  pinMode(ledPin, OUTPUT);
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected");
  Serial.println(WiFi.localIP());
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    String req = client.readStringUntil('\r');
    client.flush();

    if (req.indexOf("/led/on") != -1) {
      digitalWrite(ledPin, HIGH);
    } else if (req.indexOf("/led/off") != -1) {
      digitalWrite(ledPin, LOW);
    }

    client.println("HTTP/1.1 200 OK");
    client.println("Content-Type: text/html");
    client.println();
    client.println("<html><body>");
    client.println("<h1>Điều khiển LED</h1>");
    client.println("<form action="/led/on" method="GET"><button>BẬT</button></form>");
    client.println("<form action="/led/off" method="GET"><button>TẮT</button></form>");
    client.println("</body></html>");

    client.stop();
  }
}
```

**📌 Gợi ý mở rộng:**
- Hiển thị trạng thái LED (BẬT / TẮT)
- Làm đẹp giao diện bằng HTML/CSS

---

## ✅ Bài 2: Gửi Email cảnh báo qua IFTTT

**Mục tiêu:** Gửi email cảnh báo khi nhiệt độ vượt 30°C

**Dịch vụ cần dùng:** [IFTTT.com](https://ifttt.com)

### 📍 Các bước cấu hình:

1. Đăng nhập [https://ifttt.com](https://ifttt.com)
2. Tạo **Applet mới**:
    - IF: Webhook → nhận sự kiện tên `send_alert`
    - THEN: Gmail → Send yourself an email
3. Lấy URL webhook từ IFTTT:  
   `https://maker.ifttt.com/trigger/send_alert/with/key/YOUR_IFTTT_KEY`

---

### **Code gửi HTTP request từ ESP32 đến IFTTT**

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include "DHT.h"

#define DHTPIN 15
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

const char* ssid = "Your_SSID";
const char* password = "Your_PASSWORD";

const char* iftttEvent = "send_alert";
const char* iftttKey = "YOUR_IFTTT_KEY";

void setup() {
  Serial.begin(115200);
  dht.begin();

  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nWiFi connected");
}

void loop() {
  float temp = dht.readTemperature();

  if (temp > 30.0) {
    Serial.println("Cảnh báo: nhiệt độ cao!");

    if (WiFi.status() == WL_CONNECTED) {
      HTTPClient http;
      String url = "https://maker.ifttt.com/trigger/" + String(iftttEvent) + "/with/key/" + String(iftttKey);
      http.begin(url);
      int httpCode = http.GET();
      Serial.println("HTTP Response: " + String(httpCode));
      http.end();
    }
  } else {
    Serial.println("Nhiệt độ bình thường: " + String(temp));
  }

  delay(30000); // Kiểm tra lại sau 30s
}
```

**🎥 Gợi ý video thực hành:**
- Tạo tài khoản IFTTT
- Tạo webhook
- Test gọi thử URL bằng trình duyệt
- Upload code ESP32 và kiểm tra mail

---

**🛠 Ghi chú:**
- Thay thế `Your_SSID`, `Your_PASSWORD`, `YOUR_IFTTT_KEY` bằng thông tin thực tế.
- Đảm bảo ESP32 có kết nối WiFi ổn định.
