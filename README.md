
---

# README.md – FULL E-COMMERCE PROJECT

````markdown
# E-Commerce Full Stack Project / Dự án Thương mại Điện tử Full Stack

**Song ngữ: English / Vietnamese**

---

## 1. Giới thiệu / Introduction

Đây là dự án **e-commerce full-stack**, sát thực tế FPT Shop:

- Backend: Node.js + Express + MongoDB + TypeScript + RESTful API
- Frontend: React + TypeScript + React Hook Form + TanStack Query
- Features:  
  - User Auth (Register, Login, Google OAuth, Verify Email, Reset Password)  
  - Users Role (Admin, Customer)  
  - Product Management (CRUD, Variants, Images, Rating, Stock, Price, Sale Price, Summary, Detail)  
  - Category Management (CRUD, Image, Filter)  
  - Cart & Order Management (CRUD, Checkout, Status: pending, processing, delivered, payment completed/declined)  
  - Advanced Filters: category + rating + price range + top-selling + search + pagination + sort  

---

## 2. Cài đặt Backend / Backend Setup

### 2.1 Clone project

```bash
git clone <your-repo-url>
cd backend
npm install
````

### 2.2 Config environment variables

Tạo `.env`:

```
PORT=5000
MONGO_URI=<Your MongoDB Atlas URI>
JWT_SECRET=<YourJWTSecret>
FRONTEND_URL=<Your Frontend URL>
```

### 2.3 Seed dữ liệu mẫu

```bash
npm run seed
```

* Seed users (Admin + Customer), categories, products, carts, orders

### 2.4 Chạy local

```bash
npm run dev
```

* Server chạy trên port 5000
* Test API: `http://localhost:5000/api/products`

---

## 3. Backend API Overview

* **Auth:** `/api/auth` → register, login, google OAuth, verify email, password reset
* **Users:** `/api/users` → CRUD, update role, change password
* **Products:** `/api/products` → CRUD, filter, search, sort, pagination
* **Categories:** `/api/categories` → CRUD, get products by category + filter
* **Cart:** `/api/cart` → add/remove/update items, calculate total
* **Orders:** `/api/orders` → create order, status tracking, filter, history

**All endpoints** follow **RESTful API** standard.

---

## 4. Deploy Backend trên Render / Deploy Backend on Render

1. Push code lên GitHub
2. Render → New Web Service → Connect GitHub
3. Build Command: `npm install && npm run build`
4. Start Command: `npm run start`
5. Env Variables: copy từ `.env`
6. Test: `https://your-backend.onrender.com/api/products`

---

## 5. Cài đặt Frontend / Frontend Setup

```bash
cd frontend
npm install
```

### 5.1 Config environment variables

Tạo `.env.local`:

```
VITE_API_URL=https://your-backend.onrender.com/api
```

### 5.2 Chạy local

```bash
npm run dev
```

* Frontend sẽ chạy local trên `http://localhost:5173` (Vite default)

---

## 6. Frontend Features

* **Auth Flow:** register → verify email → login
* **Product Listing:** advanced filter, search, sort, pagination
* **Cart:** add, update, remove items
* **Checkout:** create order, payment status
* **Order History:** track order status
* **Technologies:** React + TypeScript + React Hook Form + TanStack Query

---

## 7. React Components Overview

| Component             | Function                                      |
| --------------------- | --------------------------------------------- |
| `Register`            | Form đăng ký người dùng                       |
| `Login`               | Form đăng nhập                                |
| `ProductListAdvanced` | Hiển thị sản phẩm, filter, search, pagination |
| `Cart`                | Quản lý giỏ hàng                              |
| `Orders`              | Xem lịch sử đơn hàng                          |
| `App.tsx`             | Router chính, định tuyến tất cả page          |

---

## 8. Advanced Filters

* Filter kết hợp: category + rating + top-selling + price min/max
* Search kết hợp filter
* Sort: priceAsc, priceDesc, rating, newest
* Pagination hỗ trợ multi-page

**Example:**
`GET /api/products?categoryId=xxx&rating=4&minPrice=500&maxPrice=1500&topSelling=true&page=2&limit=10&sortBy=priceDesc`

---

## 9. Full Flow Example

1. User đăng ký → verify email → login
2. Browse products → filter/search → add to cart
3. Update quantity/remove items
4. Checkout → tạo đơn hàng
5. Xem lịch sử đơn hàng + trạng thái

**Frontend sử dụng:** React Hook Form + TanStack Query
**Backend:** RESTful API + JWT Auth

---

## 10. Deploy Frontend trên Vercel / Netlify

1. Vercel → New Project → Import GitHub Repo
2. Framework: React + TypeScript + Vite
3. Env: `VITE_API_URL=https://your-backend.onrender.com/api`
4. Build Command: `npm run build`
5. Output Directory: `dist`
6. Deploy → Test Frontend URL

---

## 11. Testing API

* Có thể dùng **Postman / Insomnia**
* Collection mẫu:

| Endpoint             | Method | Body / Params                                                                | Description       |
| -------------------- | ------ | ---------------------------------------------------------------------------- | ----------------- |
| `/api/auth/register` | POST   | username, email, password, phone, dob                                        | Đăng ký           |
| `/api/auth/login`    | POST   | email, password                                                              | Đăng nhập         |
| `/api/products`      | GET    | filters: categoryId, rating, minPrice, maxPrice, search, sortBy, page, limit | Lấy sản phẩm      |
| `/api/cart/add`      | POST   | productId, quantity                                                          | Thêm vào giỏ hàng |
| `/api/orders`        | POST   | paymentMethod                                                                | Tạo đơn hàng      |

---

## 12. Cấu trúc thư mục

```
backend/
  src/
    models/
    controllers/
    routes/
    middlewares/
    seed/
    app.ts
frontend/
  src/
    api/
    components/
    App.tsx
    main.tsx
```

---

## 13. Technologies

* Backend: Node.js, Express, MongoDB, TypeScript
* Frontend: React, TypeScript, React Hook Form, TanStack Query, Vite
* Deploy: Render (backend), Vercel/Netlify (frontend)

---

## 14. Notes / Lưu ý

* Backend API RESTful, JWT Auth
* Frontend sử dụng TanStack Query cache + React Hook Form validation
* Full stack flow giống FPT Shop
* Seed data có thể chỉnh sửa để test nhiều scenario

---

