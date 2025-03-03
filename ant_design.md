Dưới đây là một loạt bài tập từ đơn giản đến nâng cao sử dụng Ant Design với React. Mỗi bài tập sẽ kèm theo hướng dẫn cài đặt môi trường, các lệnh cần thiết và mã nguồn mẫu. Bạn có thể làm theo thứ tự để từ từ nắm bắt và mở rộng kiến thức.

---

## 1. Cài Đặt Môi Trường

Mở terminal (CMD, PowerShell hoặc Terminal trên macOS/Linux) và chạy các lệnh sau:

1. **Tạo dự án React mới:**
   ```bash
   npx create-react-app ant-design-app
   cd ant-design-app
   ```

2. **Cài đặt Ant Design:**
   ```bash
   npm install antd
   ```
   
3. **(Bài tập nâng cao) Cài đặt Axios để gọi API:**
   ```bash
   npm install axios
   ```

4. **(Bài tập nâng cao) Cài đặt JSON Server (dùng làm API giả lập):**
   ```bash
   npm install -g json-server
   ```

---

## 2. Bài Tập Đơn Giản: “Hello Ant Design”

### Mục tiêu:
- Hiển thị một giao diện đơn giản với **Header**, **Content** và **Footer** từ Ant Design.

### Hướng dẫn & Mã nguồn:
1. Mở file `src/App.js` và thay thế nội dung bằng mã sau:
   ```jsx
   import React from 'react';
   import { Layout } from 'antd';
   import 'antd/dist/reset.css'; // Hoặc dùng 'antd/dist/antd.css'

   const { Header, Content, Footer } = Layout;

   function App() {
     return (
       <Layout>
         <Header style={{ color: 'white' }}>
           Header - Chào mừng đến với Ant Design
         </Header>
         <Content style={{ padding: '20px' }}>
           <h1>Hello Ant Design!</h1>
           <p>Đây là ứng dụng React đầu tiên của bạn với Ant Design.</p>
         </Content>
         <Footer style={{ textAlign: 'center' }}>
           Ant Design App ©2025
         </Footer>
       </Layout>
     );
   }

   export default App;
   ```
2. **Chạy ứng dụng:**
   ```bash
   npm start
   ```
   Trình duyệt sẽ mở ra trang hiển thị giao diện với Header, nội dung và Footer đơn giản.

---

## 3. Bài Tập Trung Cấp: Ứng Dụng Quản Lý Người Dùng

### Mục tiêu:
- Hiển thị danh sách người dùng với **Table**.
- Thêm người dùng mới qua **Form** trong **Modal**.
- Xóa người dùng khỏi danh sách.

### Hướng dẫn & Mã nguồn:
1. Mở file `src/App.js` và thay thế nội dung bằng đoạn mã sau:
   ```jsx
   import React, { useState } from 'react';
   import { Layout, Table, Form, Input, Button, Modal, Space } from 'antd';
   import 'antd/dist/reset.css';

   const { Header, Content, Footer } = Layout;

   // Dữ liệu mẫu ban đầu
   const initialUsers = [
     { key: '1', name: 'Nguyễn Văn A', email: 'a@gmail.com' },
     { key: '2', name: 'Trần Thị B', email: 'b@gmail.com' },
   ];

   function App() {
     const [users, setUsers] = useState(initialUsers);
     const [isModalVisible, setIsModalVisible] = useState(false);
     const [form] = Form.useForm();

     // Hiển thị Modal thêm người dùng
     const showModal = () => setIsModalVisible(true);

     // Xử lý khi nhấn OK trong Modal
     const handleOk = () => {
       form.validateFields()
         .then(values => {
           form.resetFields();
           setUsers([...users, { key: Date.now().toString(), ...values }]);
           setIsModalVisible(false);
         })
         .catch(info => {
           console.log('Validate Failed:', info);
         });
     };

     // Đóng Modal
     const handleCancel = () => setIsModalVisible(false);

     // Xóa người dùng theo key
     const deleteUser = key => {
       setUsers(users.filter(user => user.key !== key));
     };

     // Định nghĩa các cột cho bảng
     const columns = [
       { title: 'Tên người dùng', dataIndex: 'name', key: 'name' },
       { title: 'Email', dataIndex: 'email', key: 'email' },
       { 
         title: 'Hành động', 
         key: 'action', 
         render: (_, record) => (
           <Space>
             <Button danger onClick={() => deleteUser(record.key)}>Xóa</Button>
           </Space>
         ) 
       },
     ];

     return (
       <Layout>
         <Header style={{ color: 'white', fontSize: '20px' }}>Quản lý người dùng</Header>
         <Content style={{ padding: '20px' }}>
           <Button type="primary" onClick={showModal} style={{ marginBottom: '20px' }}>
             Thêm người dùng
           </Button>
           <Table dataSource={users} columns={columns} />
           <Modal 
             title="Thêm người dùng" 
             open={isModalVisible} 
             onOk={handleOk} 
             onCancel={handleCancel}
           >
             <Form form={form} layout="vertical">
               <Form.Item 
                 name="name" 
                 label="Tên người dùng" 
                 rules={[{ required: true, message: 'Vui lòng nhập tên người dùng!' }]}
               >
                 <Input placeholder="Nhập tên người dùng" />
               </Form.Item>
               <Form.Item 
                 name="email" 
                 label="Email" 
                 rules={[{ required: true, type: 'email', message: 'Vui lòng nhập email hợp lệ!' }]}
               >
                 <Input placeholder="Nhập email" />
               </Form.Item>
             </Form>
           </Modal>
         </Content>
         <Footer style={{ textAlign: 'center' }}>
           Ant Design - Bài tập quản lý người dùng
         </Footer>
       </Layout>
     );
   }

   export default App;
   ```
2. **Chạy ứng dụng:**
   ```bash
   npm start
   ```
   Giao diện sẽ hiển thị danh sách người dùng, cho phép thêm mới qua Modal và xóa người dùng.

---

## 4. Bài Tập Nâng Cao: Ứng Dụng Quản Lý Sản Phẩm Có Tích Hợp API

### Mục tiêu:
- Hiển thị danh sách sản phẩm sử dụng **Table** có phân trang, sắp xếp.
- Thêm, chỉnh sửa và xóa sản phẩm thông qua **Modal** và **Form**.
- Tích hợp API giả lập bằng **JSON Server** (với Axios) để quản lý dữ liệu.

### Bước 1: Thiết lập API giả lập với JSON Server
1. **Tạo file `db.json`** trong thư mục gốc của dự án với nội dung:
   ```json
   {
     "products": [
       { "id": 1, "name": "Sản phẩm A", "price": 100 },
       { "id": 2, "name": "Sản phẩm B", "price": 200 }
     ]
   }
   ```
2. **Chạy JSON Server:**
   ```bash
   json-server --watch db.json --port 3001
   ```
   API sẽ được phục vụ tại địa chỉ `http://localhost:3001/products`.

### Bước 2: Xây dựng giao diện quản lý sản phẩm

1. Mở file `src/App.js` và thay thế nội dung bằng đoạn mã sau:
   ```jsx
   import React, { useState, useEffect } from 'react';
   import { Layout, Table, Button, Modal, Form, Input, InputNumber, Space } from 'antd';
   import axios from 'axios';
   import 'antd/dist/reset.css';

   const { Header, Content, Footer } = Layout;

   function App() {
     const [products, setProducts] = useState([]);
     const [isModalVisible, setIsModalVisible] = useState(false);
     const [editingProduct, setEditingProduct] = useState(null);
     const [form] = Form.useForm();

     const apiUrl = 'http://localhost:3001/products';

     // Lấy danh sách sản phẩm từ API
     const fetchProducts = async () => {
       try {
         const res = await axios.get(apiUrl);
         setProducts(res.data);
       } catch (error) {
         console.error('Error fetching products:', error);
       }
     };

     useEffect(() => {
       fetchProducts();
     }, []);

     // Hiển thị Modal thêm mới sản phẩm
     const showAddModal = () => {
       setEditingProduct(null);
       form.resetFields();
       setIsModalVisible(true);
     };

     // Hiển thị Modal chỉnh sửa sản phẩm
     const showEditModal = product => {
       setEditingProduct(product);
       form.setFieldsValue(product);
       setIsModalVisible(true);
     };

     const handleOk = async () => {
       try {
         const values = await form.validateFields();
         if (editingProduct) {
           // Cập nhật sản phẩm
           await axios.put(`${apiUrl}/${editingProduct.id}`, values);
         } else {
           // Thêm sản phẩm mới
           await axios.post(apiUrl, values);
         }
         fetchProducts();
         setIsModalVisible(false);
         form.resetFields();
       } catch (error) {
         console.error('Validation Failed:', error);
       }
     };

     const handleCancel = () => {
       setIsModalVisible(false);
       form.resetFields();
     };

     // Xóa sản phẩm
     const deleteProduct = async id => {
       try {
         await axios.delete(`${apiUrl}/${id}`);
         fetchProducts();
       } catch (error) {
         console.error('Error deleting product:', error);
       }
     };

     const columns = [
       { title: 'Tên sản phẩm', dataIndex: 'name', key: 'name' },
       { 
         title: 'Giá', 
         dataIndex: 'price', 
         key: 'price',
         render: price => `$${price}`
       },
       { 
         title: 'Hành động', 
         key: 'action', 
         render: (_, record) => (
           <Space>
             <Button type="primary" onClick={() => showEditModal(record)}>Sửa</Button>
             <Button danger onClick={() => deleteProduct(record.id)}>Xóa</Button>
           </Space>
         ) 
       },
     ];

     return (
       <Layout>
         <Header style={{ color: 'white', fontSize: '20px' }}>
           Quản lý Sản phẩm
         </Header>
         <Content style={{ padding: '20px' }}>
           <Button type="primary" onClick={showAddModal} style={{ marginBottom: '20px' }}>
             Thêm sản phẩm
           </Button>
           <Table 
             dataSource={products} 
             columns={columns} 
             rowKey="id" 
             pagination={{ pageSize: 5 }}
           />
           <Modal 
             title={editingProduct ? "Chỉnh sửa sản phẩm" : "Thêm sản phẩm"} 
             open={isModalVisible} 
             onOk={handleOk} 
             onCancel={handleCancel}
           >
             <Form form={form} layout="vertical">
               <Form.Item 
                 name="name" 
                 label="Tên sản phẩm" 
                 rules={[{ required: true, message: 'Vui lòng nhập tên sản phẩm!' }]}
               >
                 <Input placeholder="Nhập tên sản phẩm" />
               </Form.Item>
               <Form.Item 
                 name="price" 
                 label="Giá" 
                 rules={[{ required: true, message: 'Vui lòng nhập giá sản phẩm!' }]}
               >
                 <InputNumber style={{ width: '100%' }} placeholder="Nhập giá sản phẩm" />
               </Form.Item>
             </Form>
           </Modal>
         </Content>
         <Footer style={{ textAlign: 'center' }}>
           Ant Design - Ứng dụng Quản lý sản phẩm
         </Footer>
       </Layout>
     );
   }

   export default App;
   ```
2. **Chạy JSON Server (ở thư mục chứa `db.json`):**
   ```bash
   json-server --watch db.json --port 3001
   ```
3. **Chạy ứng dụng React:**
   ```bash
   npm start
   ```
   Bạn sẽ có giao diện quản lý sản phẩm với khả năng thêm, chỉnh sửa, xóa sản phẩm; dữ liệu được lưu và cập nhật qua API giả lập.

---

## Tổng Kết

- **Bài tập đơn giản:** Giúp bạn làm quen với cấu trúc Layout của Ant Design.
- **Bài tập trung cấp:** Xây dựng ứng dụng quản lý người dùng với các thao tác thêm và xóa sử dụng Table, Form và Modal.
- **Bài tập nâng cao:** Mở rộng ứng dụng quản lý sản phẩm tích hợp API giả lập với JSON Server, sử dụng Axios để xử lý dữ liệu, đồng thời thêm tính năng chỉnh sửa và phân trang.

Hãy thử từng bước một và mở rộng thêm các tính năng nếu cần. Chúc bạn học tập và lập trình thành công!
