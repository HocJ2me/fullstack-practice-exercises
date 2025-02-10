## 📅 Bài Tập Tuần Này: Xây Dựng Hệ Thống Xác Thực JWT và Quản Lý Người Dùng

### 🌟 **Yêu cầu bài tập**
1. Thêm xác thực **JWT (JSON Web Token)** để bảo vệ API và giữ người dùng đăng nhập.
2. Tạo trang **Dashboard** hiển thị thông tin người dùng sau khi đăng nhập.
3. Thêm tính năng **đăng xuất (logout)**.
4. Nâng cấp giao diện với **CSS** để trang web trông chuyên nghiệp hơn.

---

### 📌 **Công nghệ sử dụng**
- **Frontend**: HTML, CSS (hoặc Bootstrap để làm đẹp giao diện), JavaScript.
- **Backend**: Node.js, Express.js.
- **Database**: MySQL.
- **Bảo mật**: JWT (JSON Web Token).

---

## 🛠 **Hướng dẫn chi tiết**

### 1️⃣ **Cài đặt thêm thư viện JWT**

Trong thư mục dự án, cài đặt thêm thư viện JWT:
```sh
npm install jsonwebtoken
```

---

### 2️⃣ **Cập nhật Backend (server.js)**

1. **Thêm JWT vào API đăng nhập** để tạo token khi người dùng đăng nhập thành công.
2. **Tạo middleware xác thực** để kiểm tra token khi truy cập vào dashboard.

#### Cập nhật **server.js**:
```javascript
const jwt = require('jsonwebtoken'); // Thêm dòng này

// Thêm API Đăng nhập mới với JWT
app.post('/login', (req, res) => {
    const { email, password } = req.body;
    const sql = "SELECT * FROM users WHERE email = ?";
    
    db.query(sql, [email], (err, results) => {
        if (err) return res.status(500).json({ error: 'Lỗi database' });
        if (results.length === 0) return res.status(401).json({ error: 'Email không tồn tại' });

        bcrypt.compare(password, results[0].password, (err, isMatch) => {
            if (err) return res.status(500).json({ error: 'Lỗi so sánh mật khẩu' });
            if (!isMatch) return res.status(401).json({ error: 'Mật khẩu không đúng' });

            // Tạo JWT Token
            const token = jwt.sign({ id: results[0].id, username: results[0].username }, 'secret_key', { expiresIn: '1h' });
            res.json({ message: 'Đăng nhập thành công!', token });
        });
    });
});

// Middleware xác thực token
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

// API Dashboard (Yêu cầu phải đăng nhập)
app.get('/dashboard', authenticateToken, (req, res) => {
    res.json({ message: `Chào mừng ${req.user.username} đến với Dashboard!` });
});
```

---

### 3️⃣ **Cập nhật Frontend**

#### **index.html**:
Thêm liên kết để chuyển sang **Dashboard** sau khi đăng nhập thành công.
```html
<h2>Đăng nhập</h2>
<input type="email" id="login-email" placeholder="Email">
<input type="password" id="login-password" placeholder="Mật khẩu">
<button onclick="login()">Đăng nhập</button>
<button onclick="logout()">Đăng xuất</button>

<a href="dashboard.html">Vào Dashboard</a>
```

---

#### **Tạo file dashboard.html**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Dashboard</title>
</head>
<body>
    <h2>Dashboard</h2>
    <div id="dashboard-content"></div>
    <button onclick="logout()">Đăng xuất</button>

    <script src="script.js"></script>
</body>
</html>
```

---

#### **Cập nhật script.js**:
```javascript
const API_URL = "http://localhost:3000";

// Lưu token khi đăng nhập thành công
function login() {
    const email = document.getElementById("login-email").value;
    const password = document.getElementById("login-password").value;

    fetch(`${API_URL}/login`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password })
    })
    .then(res => res.json())
    .then(data => {
        if (data.token) {
            localStorage.setItem("token", data.token);
            alert(data.message);
            window.location.href = "dashboard.html";
        } else {
            alert(data.error);
        }
    })
    .catch(err => console.error(err));
}

// Lấy dữ liệu từ dashboard với token
function loadDashboard() {
    const token = localStorage.getItem("token");
    if (!token) {
        alert("Bạn chưa đăng nhập!");
        window.location.href = "index.html";
        return;
    }

    fetch(`${API_URL}/dashboard`, {
        headers: { "Authorization": `Bearer ${token}` }
    })
    .then(res => res.json())
    .then(data => {
        document.getElementById("dashboard-content").innerText = data.message;
    })
    .catch(err => {
        alert("Phiên đăng nhập hết hạn, vui lòng đăng nhập lại.");
        window.location.href = "index.html";
    });
}

// Hàm đăng xuất
function logout() {
    localStorage.removeItem("token");
    alert("Đăng xuất thành công!");
    window.location.href = "index.html";
}

// Tự động load dashboard nếu đang ở dashboard.html
if (window.location.pathname.includes("dashboard.html")) {
    loadDashboard();
}
```

---

### ✅ **Hoàn thành**
1. Chạy backend: `node server.js`.
2. Mở `index.html` để đăng nhập.
3. Sau khi đăng nhập thành công, bạn sẽ được chuyển đến `dashboard.html` với thông tin người dùng.
4. Test tính năng **đăng xuất** và kiểm tra xem có vào được dashboard mà không đăng nhập hay không.

---

### 🌟 **Thử thách thêm (Không bắt buộc)**:
1. Thêm tính năng **quên mật khẩu**.
2. Thêm **Bootstrap** để cải thiện giao diện đẹp mắt hơn.
3. Thêm chức năng phân quyền (admin, user).

---

Chúc bạn làm bài tập vui vẻ! Nếu gặp khó khăn phần nào mình luôn sẵn sàng giúp nhé. 😊

