
---

## 📅 Bài Tập Tuần Này: Xây Dựng Hệ THỐNG CHAT REALTIME với Socket.io, Node.js và MySQL

### 🌟 **Yêu cầu bài tập**
1. **Giao diện chat realtime:**  
   - Tạo giao diện cho phép người dùng gửi và nhận tin nhắn theo thời gian thực.
2. **Tích hợp Socket.io:**  
   - Thiết lập kết nối realtime giữa client và server bằng Socket.io.
3. **Đăng nhập và cấp phát JWT:**  
   - Xây dựng API đăng nhập để người dùng xác thực thông qua MySQL và nhận token JWT.
4. **Bảo vệ API và kết nối realtime:**  
   - Sử dụng middleware xác thực JWT cho các API (ví dụ: lấy thông tin người dùng) và cho kết nối Socket.io.
5. **Lưu lịch sử chat vào MySQL (Thử thách):**  
   - Khi người dùng gửi tin nhắn, lưu tin nhắn đó vào bảng “messages” trong MySQL.
6. **Giao diện chuyên nghiệp:**  
   - Sử dụng CSS/Bootstrap để giao diện trở nên hiện đại và thân thiện với người dùng.

---

### 📌 **Công nghệ sử dụng**
- **Frontend:** HTML, CSS (hoặc Bootstrap), JavaScript.
- **Backend:** Node.js, Express.js.
- **Realtime:** Socket.io.
- **Database:** MySQL.
- **Bảo mật:** JWT (JSON Web Token).

---

## 🛠 **Hướng dẫn chi tiết**

### 1️⃣ **Cài đặt thư viện cần thiết**

Mở terminal và chạy lệnh sau:
```sh
npm install express socket.io jsonwebtoken mysql2
```

---

### 2️⃣ **Cập nhật Backend (server.js)**

Tạo file **server.js** với nội dung dưới đây. Phiên bản này có thêm xử lý lưu tin nhắn vào MySQL (bảng “messages”). Bạn cần tạo bảng “users” và “messages” trong cơ sở dữ liệu của bạn.

Ví dụ SQL tạo bảng messages:
```sql
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255),
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### File **server.js**:
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');
const mysql = require('mysql2');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(express.json());

// Kết nối MySQL (cập nhật thông tin của bạn)
const db = mysql.createPool({
    host: 'localhost',
    user: 'your_mysql_user',
    password: 'your_mysql_password',
    database: 'chatapp'
});

// API đăng nhập
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    // Giả định xác thực: kiểm tra username và password trong bảng users
    const sql = "SELECT * FROM users WHERE username = ? AND password = ?";
    db.query(sql, [username, password], (err, results) => {
        if (err) return res.status(500).json({ error: 'Lỗi database' });
        if (results.length === 0) return res.status(401).json({ error: 'Thông tin đăng nhập không hợp lệ' });

        // Tạo JWT Token
        const token = jwt.sign({ id: results[0].id, username: results[0].username }, 'secret_key', { expiresIn: '1h' });
        res.json({ message: 'Đăng nhập thành công!', token });
    });
});

// Middleware xác thực token cho API
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'Token không tồn tại' });

    jwt.verify(token, 'secret_key', (err, user) => {
        if (err) return res.status(403).json({ error: 'Token không hợp lệ hoặc đã hết hạn' });
        req.user = user;
        next();
    });
}

// API ví dụ: lấy thông tin người dùng (profile)
app.get('/profile', authenticateToken, (req, res) => {
    const sql = "SELECT id, username FROM users WHERE id = ?";
    db.query(sql, [req.user.id], (err, results) => {
        if (err) return res.status(500).json({ error: 'Lỗi database' });
        if (results.length === 0) return res.status(404).json({ error: 'Không tìm thấy người dùng' });
        res.json({ message: `Chào mừng ${results[0].username}!` });
    });
});

// Xác thực token cho kết nối Socket.io
io.use((socket, next) => {
    const token = socket.handshake.query.token;
    if (!token) return next(new Error('Authentication error'));
    jwt.verify(token, 'secret_key', (err, user) => {
        if (err) return next(new Error('Authentication error'));
        socket.user = user;
        next();
    });
});

// Xử lý kết nối realtime với Socket.io
io.on('connection', (socket) => {
    console.log(`User ${socket.user.username} đã kết nối`);

    // Lắng nghe sự kiện "chat message"
    socket.on('chat message', (msg) => {
        // Lưu tin nhắn vào MySQL (bảng messages)
        const sql = "INSERT INTO messages (username, message) VALUES (?, ?)";
        db.query(sql, [socket.user.username, msg], (err) => {
            if (err) console.error('Lỗi lưu tin nhắn:', err);
        });
        // Phát tin nhắn đến tất cả các client
        io.emit('chat message', { username: socket.user.username, message: msg });
    });

    socket.on('disconnect', () => {
        console.log(`User ${socket.user.username} đã ngắt kết nối`);
    });
});

server.listen(3000, () => {
    console.log('Server đang chạy trên cổng 3000');
});
```

---

### 3️⃣ **Cập nhật Frontend**

#### a. **index.html**
Tạo file **index.html** với giao diện đăng nhập và chat realtime:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Chat App</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css">
</head>
<body>
    <!-- Phần đăng nhập -->
    <div id="login-section" class="container mt-5">
        <h2>Đăng nhập</h2>
        <input type="text" id="username" placeholder="Tên đăng nhập" class="form-control mb-2">
        <input type="password" id="password" placeholder="Mật khẩu" class="form-control mb-2">
        <button onclick="login()" class="btn btn-primary">Đăng nhập</button>
    </div>

    <!-- Phần chat -->
    <div id="chat-section" class="container mt-5" style="display:none;">
        <h2>Hệ thống Chat Realtime</h2>
        <ul id="messages" class="list-group mb-3"></ul>
        <div class="input-group">
            <input id="message-input" class="form-control" placeholder="Nhập tin nhắn...">
            <div class="input-group-append">
                <button onclick="sendMessage()" class="btn btn-success">Gửi</button>
            </div>
        </div>
        <button onclick="logout()" class="btn btn-danger mt-3">Đăng xuất</button>
    </div>

    <script src="/socket.io/socket.io.js"></script>
    <script src="script.js"></script>
</body>
</html>
```

#### b. **script.js**
Tạo file **script.js** để xử lý đăng nhập, khởi tạo kết nối Socket.io và gửi tin nhắn:
```javascript
const API_URL = "http://localhost:3000";

// Hàm đăng nhập: gửi thông tin đăng nhập và lưu token nếu thành công
function login() {
    const username = document.getElementById("username").value;
    const password = document.getElementById("password").value;

    fetch(`${API_URL}/login`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username, password })
    })
    .then(res => res.json())
    .then(data => {
        if (data.token) {
            localStorage.setItem("token", data.token);
            document.getElementById("login-section").style.display = 'none';
            document.getElementById("chat-section").style.display = 'block';
            initSocket();
        } else {
            alert(data.error || "Đăng nhập thất bại!");
        }
    })
    .catch(err => console.error(err));
}

// Khởi tạo kết nối Socket.io kèm token từ localStorage
function initSocket() {
    const token = localStorage.getItem("token");
    const socket = io(API_URL, { query: { token } });

    socket.on('chat message', (data) => {
        const messages = document.getElementById("messages");
        const newMsg = document.createElement("li");
        newMsg.className = "list-group-item";
        newMsg.innerText = `${data.username}: ${data.message}`;
        messages.appendChild(newMsg);
    });

    // Gửi tin nhắn khi người dùng nhấn nút gửi
    window.sendMessage = function() {
        const input = document.getElementById("message-input");
        if (input.value.trim() !== "") {
            socket.emit('chat message', input.value);
            input.value = '';
        }
    }
}

// Hàm đăng xuất: xóa token và tải lại trang
function logout() {
    localStorage.removeItem("token");
    window.location.reload();
}
```

---

### ✅ **Hoàn thành**
1. **Backend:**  
   - Tạo bảng `users` (đã có sẵn dữ liệu để đăng nhập) và bảng `messages` trong MySQL.  
   - Chạy server bằng lệnh:  
     ```sh
     node server.js
     ```
2. **Frontend:**  
   - Mở file **index.html** trên trình duyệt.  
   - Đăng nhập với thông tin người dùng có trong bảng `users`.  
   - Khi đăng nhập thành công, giao diện chat realtime hiện ra và bạn có thể gửi, nhận tin nhắn – đồng thời các tin nhắn được lưu vào bảng `messages`.
3. **Đăng xuất:**  
   - Nhấn nút "Đăng xuất" để xóa token và tải lại trang.

---

### 🌟 **Thử thách thêm**
1. **Đăng ký người dùng:** Tạo API đăng ký với việc mã hoá mật khẩu.  
2. **Hiển thị lịch sử chat:** Tạo API lấy lịch sử tin nhắn từ MySQL và hiển thị khi người dùng đăng nhập.
3. **Tạo phòng chat:** Cho phép người dùng tạo và tham gia các phòng chat riêng.

---
