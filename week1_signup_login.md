## Bài Tập Thực Hành: Xây Dựng Web Login & Signup Cơ Bản

### 🎯 Yêu cầu bài tập
Xây dựng một trang web đơn giản cho phép người dùng:
1. Đăng ký tài khoản (signup).
2. Đăng nhập (login).
3. Hiển thị thông tin người dùng sau khi đăng nhập.

---

### 📌 Công nghệ sử dụng
- **Frontend**: HTML, CSS, JavaScript.
- **Backend**: Node.js, Express.js.
- **Database**: MySQL.

---

## 🛠 Hướng dẫn chi tiết

### 1️⃣ Thiết lập môi trường
1. Cài đặt **Node.js**: [Tải tại đây](https://nodejs.org/)
2. Cài đặt **MySQL Server**.

---

### 2️⃣ Tạo Database MySQL
```sql
CREATE DATABASE user_auth;

USE user_auth;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);
```

---

### 3️⃣ Cài đặt Node.js & Express
Mở terminal/cmd, chạy lệnh:
```sh
mkdir login-signup
cd login-signup
npm init -y
npm install express mysql2 bcryptjs cors body-parser dotenv
```

---

### 4️⃣ Viết mã backend (Node.js + Express)
Tạo file **server.js** và thêm code backend:
```javascript
require('dotenv').config();
const express = require('express');
const mysql = require('mysql2');
const bcrypt = require('bcryptjs');
const cors = require('cors');

const app = express();
app.use(express.json());
app.use(cors());

const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: '',
    database: 'user_auth'
});

db.connect(err => {
    if (err) console.error('Lỗi kết nối MySQL:', err);
    else console.log('✅ Đã kết nối MySQL');
});

// API Đăng ký
app.post('/signup', (req, res) => {
    const { username, email, password } = req.body;
    bcrypt.hash(password, 10, (err, hash) => {
        if (err) return res.status(500).json({ error: 'Lỗi mã hóa mật khẩu' });
        const sql = "INSERT INTO users (username, email, password) VALUES (?, ?, ?)";
        db.query(sql, [username, email, hash], (err, result) => {
            if (err) return res.status(500).json({ error: 'Lỗi database' });
            res.json({ message: 'Đăng ký thành công!' });
        });
    });
});

// API Đăng nhập
app.post('/login', (req, res) => {
    const { email, password } = req.body;
    const sql = "SELECT * FROM users WHERE email = ?";
    db.query(sql, [email], (err, results) => {
        if (err) return res.status(500).json({ error: 'Lỗi database' });
        if (results.length === 0) return res.status(401).json({ error: 'Email không tồn tại' });

        bcrypt.compare(password, results[0].password, (err, isMatch) => {
            if (err) return res.status(500).json({ error: 'Lỗi so sánh mật khẩu' });
            if (!isMatch) return res.status(401).json({ error: 'Mật khẩu không đúng' });
            res.json({ message: 'Đăng nhập thành công!', user: results[0] });
        });
    });
});

app.listen(3000, () => console.log('🚀 Server chạy tại http://localhost:3000'));
```

---

### 5️⃣ Viết mã frontend (HTML + JavaScript)
Tạo file **index.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Login & Signup</title>
</head>
<body>
    <h2>Đăng ký</h2>
    <input type="text" id="signup-username" placeholder="Tên đăng nhập">
    <input type="email" id="signup-email" placeholder="Email">
    <input type="password" id="signup-password" placeholder="Mật khẩu">
    <button onclick="signup()">Đăng ký</button>

    <h2>Đăng nhập</h2>
    <input type="email" id="login-email" placeholder="Email">
    <input type="password" id="login-password" placeholder="Mật khẩu">
    <button onclick="login()">Đăng nhập</button>

    <script src="script.js"></script>
</body>
</html>
```

Tạo file **script.js**:
```javascript
const API_URL = "http://localhost:3000";
function signup() {
    const username = document.getElementById("signup-username").value;
    const email = document.getElementById("signup-email").value;
    const password = document.getElementById("signup-password").value;

    fetch(`${API_URL}/signup`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username, email, password })
    })
    .then(res => res.json())
    .then(data => alert(data.message))
    .catch(err => console.error(err));
}

function login() {
    const email = document.getElementById("login-email").value;
    const password = document.getElementById("login-password").value;

    fetch(`${API_URL}/login`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password })
    })
    .then(res => res.json())
    .then(data => alert(data.message))
    .catch(err => console.error(err));
}
```

---

### ✅ Hoàn thành
- Chạy backend: `node server.js`
- Mở `index.html` trong trình duyệt và test chương trình!
