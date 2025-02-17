

## 📅 Bài Tập Tuần Này: Xây Dựng Hệ Thống Chat Realtime với Socket.io và Node.js

### 🌟 **Yêu cầu bài tập**
1. Tạo giao diện chat cho người dùng, cho phép gửi và nhận tin nhắn theo thời gian thực.
2. Tích hợp **Socket.io** để thiết lập kết nối realtime giữa client và server.
3. Xây dựng API đăng nhập để xác thực người dùng và cấp phát **JWT**.
4. Xây dựng middleware xác thực cho các API và kết nối Socket.io.
5. Nâng cấp giao diện sử dụng **CSS/Bootstrap** cho giao diện chuyên nghiệp.

---

### 📌 **Công nghệ sử dụng**
- **Frontend**: HTML, CSS (hoặc Bootstrap), JavaScript.
- **Backend**: Node.js, Express.js.
- **Realtime**: Socket.io.
- **Database**: MongoDB (hoặc MySQL nếu bạn muốn).
- **Bảo mật**: JWT (JSON Web Token) cho API đăng nhập.

---

## 🛠 **Hướng dẫn chi tiết**

### 1️⃣ **Cài đặt thư viện cần thiết**

Trong thư mục dự án, mở terminal và chạy lệnh sau để cài đặt các thư viện:
```sh
npm install express socket.io jsonwebtoken mongoose
```

---

### 2️⃣ **Cập nhật Backend (server.js)**

1. **API Đăng nhập:** Tạo API cho phép người dùng đăng nhập và nhận được JWT.
2. **Tích hợp Socket.io:** Cấu hình server để hỗ trợ kết nối realtime giữa các client.
3. **Middleware xác thực JWT:** Kiểm tra token trước khi cho phép truy cập API hoặc kết nối Socket.io.

#### Cập nhật **server.js**:
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');
const mongoose = require('mongoose');
const app = express();
const server = http.createServer(app);
const io = socketIo(server);

app.use(express.json());

// Kết nối tới MongoDB
mongoose.connect('mongodb://localhost:27017/chatapp', { useNewUrlParser: true, useUnifiedTopology: true });

// API đăng nhập
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    // Giả định xác thực thành công, thực tế bạn nên kiểm tra trong CSDL
    const token = jwt.sign({ username }, 'secret_key', { expiresIn: '1h' });
    res.json({ message: 'Đăng nhập thành công!', token });
});

// Middleware xác thực token cho API
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    if (!token) return res.sendStatus(401);
    
    jwt.verify(token, 'secret_key', (err, user) => {
        if (err) return res.sendStatus(403);
        req.user = user;
        next();
    });
}

// API kiểm tra thông tin người dùng đã đăng nhập
app.get('/profile', authenticateToken, (req, res) => {
    res.json({ message: `Chào mừng ${req.user.username}!` });
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

// Cấu hình kết nối realtime với Socket.io
io.on('connection', (socket) => {
    console.log(`User ${socket.user.username} đã kết nối`);
    
    socket.on('chat message', (msg) => {
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

#### **index.html**:
Tạo giao diện đăng nhập và giao diện chat. Khi đăng nhập thành công, hiển thị giao diện chat realtime.
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

---

#### **script.js**:
```javascript
const API_URL = "http://localhost:3000";

// Hàm đăng nhập và lưu token
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
            alert("Đăng nhập thất bại!");
        }
    })
    .catch(err => console.error(err));
}

// Khởi tạo kết nối Socket.io với token
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

    window.sendMessage = function() {
        const input = document.getElementById("message-input");
        if(input.value.trim() !== "") {
            socket.emit('chat message', input.value);
            input.value = '';
        }
    }
}

// Hàm đăng xuất
function logout() {
    localStorage.removeItem("token");
    window.location.reload();
}
```

---

### ✅ **Hoàn thành**
1. Chạy backend bằng lệnh: `node server.js`.
2. Mở file `index.html` trên trình duyệt.
3. Đăng nhập để nhận token và chuyển sang giao diện chat.
4. Gửi tin nhắn và kiểm tra tính năng realtime qua Socket.io.
5. Đăng xuất và kiểm tra lại tính năng đăng nhập.

---

### 🌟 **Thử thách thêm**
1. Lưu lịch sử chat vào cơ sở dữ liệu.
2. Thêm tính năng tạo phòng chat riêng cho các nhóm.
3. Cải thiện giao diện với thêm hiệu ứng CSS và tùy chỉnh giao diện Bootstrap.
4. Triển khai xác thực người dùng nâng cao với đăng ký, thay đổi mật khẩu và cập nhật thông tin cá nhân.

---

