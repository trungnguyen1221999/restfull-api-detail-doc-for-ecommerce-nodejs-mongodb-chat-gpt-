
---

# CHƯƠNG 14 – FULL PROJECT READY TO DEPLOY

---

## 14.1 Chuẩn bị Backend

### 14.1.1 Env file

Tạo `.env`:

```
PORT=5000
MONGO_URI=<Your MongoDB Atlas URI>
JWT_SECRET=<YourJWTSecret>
FRONTEND_URL=https://your-frontend-url.vercel.app
```

* `FRONTEND_URL` dùng cho CORS, verify email link

---

### 14.1.2 Package.json scripts

```json
"scripts": {
  "dev": "ts-node-dev src/app.ts",
  "start": "node dist/app.js",
  "build": "tsc",
  "seed": "ts-node src/seed/seedData.ts"
}
```

* `dev` → chạy local
* `build` → compile TS → JS
* `start` → chạy production
* `seed` → seed dữ liệu

---

### 14.1.3 Seed dữ liệu mẫu

```bash
npm run seed
```

* Seed users, products, categories, cart, orders mẫu

---

## 14.2 Deploy Backend lên Render

1. **Push code lên GitHub**
2. **Render → New Web Service → Connect GitHub**
3. **Environment Variables**: copy từ `.env`
4. **Build Command**: `npm install && npm run build`
5. **Start Command**: `npm run start`
6. Test API: `https://your-backend.onrender.com/api/products`

**Tip:** MongoDB Atlas phải whitelist IP `0.0.0.0/0`

---

## 14.3 Chuẩn bị Frontend

### 14.3.1 Env file

Tạo `.env.local` cho React:

```
VITE_API_URL=https://your-backend.onrender.com/api
```

* Dùng trong `axiosInstance`:

```ts
const API_URL = import.meta.env.VITE_API_URL;
```

---

### 14.3.2 Scripts package.json

```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite preview"
}
```

* `vite` template React + TypeScript

---

### 14.3.3 React Query + React Hook Form setup

* Tham khảo **CHƯƠNG 12 + 13**
* Bao gồm:

  * Register/Login
  * ProductList + Filter + Pagination
  * Cart + Checkout
  * Order History

---

## 14.4 Deploy Frontend lên Vercel

1. **Vercel → New Project → Import GitHub repo frontend**
2. **Framework**: Vite + React + TS
3. **Environment Variables**: `VITE_API_URL=https://your-backend.onrender.com/api`
4. **Build Command**: `npm run build`
5. **Output Directory**: `dist`
6. Deploy và test: `https://your-frontend-url.vercel.app`

---

## 14.5 Flow Full Stack

1. User **register** → nhận **verify email** → verify → login
2. **Browse products** → **filter/search/sort/pagination**
3. **Add to cart** → **update quantity** → **remove items**
4. **Checkout** → **create order** → **order status: pending → processing → delivered**
5. **View order history**

**Tất cả state quản lý bằng React Query**
**Forms xử lý bằng React Hook Form**
**TypeScript full stack**

---

## 14.6 Bonus – Advanced Filter + Cart + Orders Full Flow

* **Filter kết hợp nhiều điều kiện**: category + rating + price range + top-selling
* **Search kết hợp filter**
* **Pagination**
* **Add to cart từ filtered list**
* **Checkout trực tiếp**
* **Order history cập nhật theo user**

---

### ✅ Kết luận CHƯƠNG 14

* Dự án full-stack e-commerce giống FPT Shop
* Backend: Node + Express + MongoDB + TypeScript + RESTful API
* Frontend: React + TypeScript + React Hook Form + TanStack Query
* Deploy: Render (backend), Vercel/Netlify (frontend)
* Full flow: register → login → product → cart → checkout → order history
* Sử dụng **song ngữ**, **chi tiết từng bước**, copy trực tiếp lên GitHub hoặc deploy

---


