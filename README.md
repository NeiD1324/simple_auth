# simple_auth — Hướng dẫn chạy & test với Postman

Thư mục này gồm 2 ví dụ demo về xác thực với Express:

1) **Basic Authentication** (`basic_auth.js`) — bảo vệ một route bằng HTTP Basic Auth. fileciteturn1file0  
2) **Cookie-based Session** (`cookie_auth.js`) — đăng nhập, lưu token ngẫu nhiên vào MongoDB với TTL 5 phút, xác thực khi vào route bảo vệ, và xóa khi logout. fileciteturn1file1

Các dependency (Express, cookie-parser, mongoose, uuid) được khai báo trong `package.json`. fileciteturn1file2


---

## 0) Chuẩn bị môi trường

- **Node.js ≥ 18** (theo yêu cầu của Express 5).  
- **MongoDB** chạy cục bộ tại `mongodb://127.0.0.1:27017`.  
- Công cụ test REST (ví dụ: **Postman** hoặc `curl`).

Cài đặt dependency tại thư mục gốc:

```bash
npm install
```

> Nếu gặp cảnh báo về phiên bản Node, hãy cập nhật Node.js mới hơn.

---

## 1) Basic Authentication (`basic_auth.js`)

- **Server URL:** `http://localhost:3000`  
- **Route công khai:** `GET /`, `GET /public` (không cần xác thực).  
- **Route bảo vệ:** `GET /secure` (cần Basic Auth).  
- **Tài khoản demo:** `admin : 12345`. 

### Chạy server
```bash
node basic_auth.js
# Console: "Server running on http://localhost:3000"
```

### Test với Postman

1. Tạo request **GET** đến `http://localhost:3000/secure`.
2. Tab **Authorization** → chọn **Basic Auth**.
3. Username: **admin**, Password: **12345**.  
4. Gửi request → Nhận phản hồi: *"You have accessed a protected resource 🎉"*. 
<img width="1280" height="947" alt="image" src="https://github.com/user-attachments/assets/3ec0b4c1-6924-4dcd-92a1-cfa9a346a111" />

- Nếu không gửi credential → **401 Unauthorized** với `WWW-Authenticate`.   
- Nếu sai credential → **403 Access denied**. 
<img width="1300" height="983" alt="image" src="https://github.com/user-attachments/assets/08084a0f-d391-4fb6-9a7b-b994d3f546b4" />



Kiểm tra thêm route công khai:
```
http://localhost:3000/
http://localhost:3000/public
```
<img width="1292" height="947" alt="image" src="https://github.com/user-attachments/assets/153b749c-2916-4a6c-ab18-ccf86804a801" />
<img width="1286" height="955" alt="image" src="https://github.com/user-attachments/assets/83e89afe-13ef-4e44-825d-93e53aaf276b" />

## 2) Cookie-based Session (`cookie_auth.js`)

- **Server URL:** `http://localhost:3001`  
- **DB:** `mongodb://127.0.0.1:27017/cookieApp`. fileciteturn1file1  
- **User demo:** `username: admin`, `password: 12345`, role: `adsys`. fileciteturn1file1  
- **Tên cookie:** `auth_cookie_token` (HttpOnly, TTL 5 phút). fileciteturn1file1  
- **Lưu trữ trong DB:** collection `cookies` với TTL trên `createdAt`. fileciteturn1file1  
- **Route chính:**  
  - `POST /login` — xác thực user, tạo token, lưu vào DB, gửi cookie. fileciteturn1file1  
  - `GET /profile` — đọc cookie, kiểm tra DB, trả lời nếu hợp lệ. fileciteturn1file1  
  - `POST /logout` — xóa token khỏi DB và clear cookie. fileciteturn1file1

### Chạy server
```bash
# Đảm bảo MongoDB đã chạy
node cookie_auth.js
# Console: "MongoDB connected" → "Server running on http://localhost:3001"
```

### Test với Postman

#### 2.1 Đăng nhập (login)
- **POST** `http://localhost:3001/login`  
- **Body → raw → JSON**:
```json
{ "username": "admin", "password": "12345" }
```
- Kết quả: `"Logged in!"`, kèm header **Set-Cookie**.
<img width="1323" height="958" alt="image" src="https://github.com/user-attachments/assets/cf895958-ad0a-4529-8465-713ab51119fc" />

- Kiểm tra trong MongoDB: token đã được thêm vào collection `cookies`, TTL 5 phút. 
<img width="1438" height="622" alt="image" src="https://github.com/user-attachments/assets/1039bd9d-9abc-4a7e-8473-1667cf4c7e42" />

#### 2.2 Xem profile
- **GET** `http://localhost:3001/profile` (trong cùng session Postman).  
- Kết quả: `"Welcome user <id>, your cookie is valid."` 
<img width="1280" height="938" alt="image" src="https://github.com/user-attachments/assets/12757d15-c6af-4852-bb83-c4edbc23ff8a" />

Nếu cookie hết hạn hoặc bị xóa → báo lỗi **401**. 

#### 2.3 Logout
- **POST** `http://localhost:3001/logout`.  
- Kết quả: `"Logged out."`, token bị xóa trong DB, cookie được clear. 
<img width="1271" height="925" alt="image" src="https://github.com/user-attachments/assets/5ff194b9-c68c-4f9d-b110-4121f69c563e" />
<img width="1446" height="858" alt="image" src="https://github.com/user-attachments/assets/1cf7be22-5160-4962-835a-2694dab84da0" />


## 3) Xử lý sự cố

- **Port bị chiếm dụng**  
  - `basic_auth.js`: 3000, `cookie_auth.js`: 3001. 

- **MongoDB lỗi kết nối**  
  - Kiểm tra MongoDB chạy đúng trên `mongodb://127.0.0.1:27017/cookieApp`.

- **Cookie không lưu trong Postman**  
  - Gửi request login trước, sau đó mới gọi profile.  
  - Kiểm tra mục **Cookies** trong Postman.

- **Basic Auth trả về 401/403**  
  - Đảm bảo chọn **Basic Auth** với `admin/12345`. Sai → 403, thiếu header → 401.
---

## 4) Nội dung README trong repo cần có

- Giới thiệu ngắn gọn 2 demo.  
- Lệnh chạy server.  
- Cách test bằng Postman và ví dụ request.  
- Lệnh curl thay thế.  
- Ghi chú về MongoDB & TTL cookie.

Chúc bạn test thành công!
