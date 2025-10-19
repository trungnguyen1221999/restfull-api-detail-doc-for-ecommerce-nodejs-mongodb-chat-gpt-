
---

# CHƯƠNG 10 – DEPLOY BACKEND LÊN RENDER

---

## 10.1 Chuẩn bị dự án

1. Đảm bảo **project có package.json** với script:

```json
"scripts": {
  "dev": "ts-node-dev src/app.ts",
  "start": "node dist/app.js",
  "build": "tsc",
  "seed": "ts-node src/seed/seedData.ts"
}
```

2. `.env` phải có:

```
PORT=5000
MONGO_URI=<Your MongoDB Atlas URI>
JWT_SECRET=<YourJWTSecret>
```

---

## 10.2 Build và test locally

```bash
npm install
npm run seed       # seed data
npm run dev        # test locally
```

* Kiểm tra các API `/api/auth/register`, `/api/products`, `/api/categories`
* Confirm MongoDB Atlas đang kết nối

---

## 10.3 Đẩy code lên GitHub

* Tạo repo mới
* Push toàn bộ project TypeScript lên GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <github-repo-url>
git push -u origin main
```

---

## 10.4 Deploy lên Render

1. **Tạo Web Service mới** trên Render:

* `New → Web Service → Connect GitHub Repo`
* Branch: `main`
* Environment: Node
* Build Command: `npm install && npm run build`
* Start Command: `npm run start`
* Environment Variables: copy từ `.env`

2. Render sẽ tự động build và deploy.
3. Test URL public, ví dụ: `https://your-backend.onrender.com/api/products`

**Tip:** nếu dùng MongoDB Atlas, đảm bảo IP whitelist `0.0.0.0/0` để Render có thể kết nối.

---

# CHƯƠNG 11 – FRONTEND GỌI API (TypeScript + React)

---

## 11.1 Setup React + TypeScript

```bash
npx create-react-app frontend --template typescript
cd frontend
npm install axios react-router-dom
```

---

## 11.2 Axios Instance / src/api/axios.ts

```ts
import axios from "axios";

const API_URL = "https://your-backend.onrender.com/api";

const axiosInstance = axios.create({
  baseURL: API_URL,
  headers: { "Content-Type": "application/json" },
});

export default axiosInstance;
```

---

## 11.3 Auth API / src/api/auth.ts

```ts
import axiosInstance from "./axios";

interface RegisterData {
  username: string;
  email: string;
  password: string;
  avatar?: string;
  dob?: string;
  phone?: string;
}

export const register = async (data: RegisterData) => {
  const res = await axiosInstance.post("/auth/register", data);
  return res.data;
};

export const login = async (email: string, password: string) => {
  const res = await axiosInstance.post("/auth/login", { email, password });
  return res.data;
};
```

---

## 11.4 Product API / src/api/product.ts

```ts
import axiosInstance from "./axios";

export interface ProductFilter {
  search?: string;
  categoryId?: string;
  minPrice?: number;
  maxPrice?: number;
  rating?: number;
  topSelling?: boolean;
  sortBy?: "priceAsc" | "priceDesc" | "rating" | "newest";
  page?: number;
  limit?: number;
}

export const getProducts = async (filters?: ProductFilter) => {
  const res = await axiosInstance.get("/products", { params: filters });
  return res.data;
};

export const getProductById = async (id: string) => {
  const res = await axiosInstance.get(`/products/${id}`);
  return res.data;
};
```

---

## 11.5 Category API / src/api/category.ts

```ts
import axiosInstance from "./axios";

export const getCategories = async () => {
  const res = await axiosInstance.get("/categories");
  return res.data;
};

export const getProductsByCategory = async (categoryId: string, filters?: any) => {
  const res = await axiosInstance.get(`/categories/${categoryId}/products`, { params: filters });
  return res.data;
};
```

---

## 11.6 Cart API / src/api/cart.ts

```ts
import axiosInstance from "./axios";

export const getCart = async () => {
  const res = await axiosInstance.get("/cart");
  return res.data;
};

export const addToCart = async (productId: string, quantity: number) => {
  const res = await axiosInstance.post("/cart/add", { productId, quantity });
  return res.data;
};

export const removeFromCart = async (productId: string) => {
  const res = await axiosInstance.delete(`/cart/remove/${productId}`);
  return res.data;
};

export const updateCartItem = async (productId: string, quantity: number) => {
  const res = await axiosInstance.put("/cart/update", { productId, quantity });
  return res.data;
};
```

---

## 11.7 Order API / src/api/order.ts

```ts
import axiosInstance from "./axios";

export const createOrder = async (paymentMethod: string) => {
  const res = await axiosInstance.post("/orders", { paymentMethod });
  return res.data;
};

export const getUserOrders = async () => {
  const res = await axiosInstance.get("/orders/my-orders");
  return res.data;
};
```

---

## 11.8 Sử dụng trong component React

```tsx
import React, { useEffect, useState } from "react";
import { getProducts } from "../api/product";

const ProductsList: React.FC = () => {
  const [products, setProducts] = useState<any[]>([]);

  useEffect(() => {
    const fetchProducts = async () => {
      const data = await getProducts({ search: "iphone", minPrice: 500, maxPrice: 1500 });
      setProducts(data);
    };
    fetchProducts();
  }, []);

  return (
    <div>
      {products.map(p => (
        <div key={p._id}>
          <h3>{p.name}</h3>
          <p>{p.price} USD</p>
        </div>
      ))}
    </div>
  );
};

export default ProductsList;
```

---

### Tổng kết CHƯƠNG 11

* Frontend React + TypeScript kết nối **tất cả API backend**
* Sử dụng **Axios instance** + **TypeScript interface**
* Có thể gọi Product, Category, Cart, Order, Auth
* Filter nâng cao, pagination, search, sort đều dùng trực tiếp từ backend

---

✅ Với **CHƯƠNG 10 + 11**, bạn đã có:

1. Deploy backend lên Render
2. Frontend React/TS gọi API toàn bộ module
3. Hệ thống full-stack **RESTful e-commerce** sát thực tế FPT Shop

---

