**CHƯƠNG 1 – ENVIRONMENT SETUP** dưới dạng Markdown:

---

````markdown
# CHƯƠNG 1 – CÀI ĐẶT MÔI TRƯỜNG / CHAPTER 1 – ENVIRONMENT SETUP

Mục tiêu: Thiết lập **Node.js, TypeScript, MongoDB, và thư viện cần thiết** để xây dựng **RESTful API e-commerce**.

---

## 1.1 Cài đặt Node.js và npm / Install Node.js and npm

**Bước 1 / Step 1:** Tải Node.js từ trang chính thức:  
[https://nodejs.org/](https://nodejs.org/)  

- Chọn phiên bản **LTS** (Long Term Support).  

**Bước 2 / Step 2:** Kiểm tra cài đặt thành công:  

```bash
node -v   # Kiểm tra phiên bản Node.js
npm -v    # Kiểm tra phiên bản npm
````

**Giải thích / Explanation:**

* `node -v` → hiển thị version Node.js, ví dụ `v20.5.0`
* `npm -v` → hiển thị version npm, ví dụ `9.8.0`
* Node.js dùng để chạy backend server
* npm dùng để quản lý package/library

---

## 1.2 Tạo project Node.js / Create Node.js project

**Bước 1 / Step 1:** Tạo thư mục project:

```bash
mkdir ecommerce-api
cd ecommerce-api
```

**Bước 2 / Step 2:** Khởi tạo npm project:

```bash
npm init -y
```

**JSON mẫu / Sample `package.json`:**

```json
{
  "name": "ecommerce-api",
  "version": "1.0.0",
  "main": "dist/server.js",
  "scripts": {
    "dev": "ts-node-dev src/server.ts",
    "build": "tsc"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

**Giải thích / Explanation:**

* `"dev"` script → chạy server bằng `ts-node-dev`
* `"build"` script → biên dịch TypeScript sang JavaScript

---

## 1.3 Cài đặt các thư viện cần thiết / Install dependencies

**Bước 1 / Step 1:** Cài dependencies chính:

```bash
npm install express mongoose dotenv bcrypt jsonwebtoken cors
```

**Giải thích / Explanation:**

* `express` → framework backend
* `mongoose` → ORM kết nối MongoDB
* `dotenv` → quản lý biến môi trường `.env`
* `bcrypt` → mã hóa password
* `jsonwebtoken` → tạo JWT token
* `cors` → cho phép frontend từ domain khác truy cập API

**Bước 2 / Step 2:** Cài TypeScript và tools hỗ trợ:

```bash
npm install --save-dev typescript ts-node-dev @types/node @types/express @types/cors
```

**Giải thích / Explanation:**

* `typescript` → ngôn ngữ tĩnh, mạnh mẽ
* `ts-node-dev` → chạy TypeScript trực tiếp mà không cần build
* `@types/...` → type declaration cho TypeScript

---

## 1.4 Tạo file cấu hình TypeScript / Create TypeScript config

```bash
npx tsc --init
```

**tsconfig.json cơ bản / Basic config:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

**Giải thích / Explanation:**

* `"target": "ES2020"` → biên dịch sang ES2020
* `"module": "commonjs"` → chuẩn module Node.js
* `"outDir": "./dist"` → thư mục chứa JS sau khi build
* `"rootDir": "./src"` → thư mục source TypeScript
* `"strict": true` → bật kiểm tra type nghiêm ngặt
* `"esModuleInterop": true` → hỗ trợ import default

---

## 1.5 Tạo file .env / Create .env file

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ecommerce
JWT_SECRET=your_secret_key
```

**Giải thích / Explanation:**

* `PORT` → port server chạy
* `MONGO_URI` → URI kết nối MongoDB
* `JWT_SECRET` → secret key để tạo JWT token

---

## 1.6 Tạo file server / app / Create server/app file

**src/server.ts:**

```ts
import express from "express";
import dotenv from "dotenv";
import { connectDB } from "./config/db";

dotenv.config();
connectDB();

const app = express();
app.use(express.json()); // middleware parse JSON

app.get("/", (req, res) => {
  res.send("API is running...");
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

**Giải thích / Explanation:**

* `express.json()` → parse body request JSON
* `/` → endpoint test server
* `connectDB()` → kết nối MongoDB
* `app.listen(PORT)` → chạy server

---

## ✅ Kết luận Chương 1 / Conclusion Chapter 1

Sau khi hoàn thành **Chương 1**, bạn đã có:

1. Node.js + npm + TypeScript + ts-node-dev
2. Thư mục project cơ bản
3. Cài đặt đầy đủ dependencies
4. File `.env` chứa biến môi trường
5. File server cơ bản chạy được
6. Kết nối MongoDB thành công


