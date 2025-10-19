 **CHƯƠNG 2 – CẤU TRÚC THƯ MỤC & MVC PATTERN** 

```markdown
# CHƯƠNG 2 – CẤU TRÚC THƯ MỤC & MVC PATTERN / CHAPTER 2 – FOLDER STRUCTURE & MVC PATTERN

Mục tiêu: Thiết kế **cấu trúc thư mục chuẩn**, theo mô hình **MVC (Model-View-Controller)**, dễ mở rộng cho **RESTful API e-commerce**.

---

## 2.1 Cấu trúc thư mục / Folder Structure

Cấu trúc đề xuất:

```

ecommerce-api/
├── src/
│   ├── config/
│   │   └── db.ts
│   ├── controllers/
│   │   ├── user.controller.ts
│   │   ├── product.controller.ts
│   │   ├── category.controller.ts
│   │   └── order.controller.ts
│   ├── models/
│   │   ├── user.model.ts
│   │   ├── product.model.ts
│   │   ├── category.model.ts
│   │   └── order.model.ts
│   ├── routes/
│   │   ├── user.routes.ts
│   │   ├── product.routes.ts
│   │   ├── category.routes.ts
│   │   └── order.routes.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   └── error.middleware.ts
│   ├── utils/
│   │   └── helpers.ts
│   ├── app.ts
│   └── server.ts
├── package.json
├── tsconfig.json
└── .env

```

**Giải thích / Explanation:**

- `src/config/` → chứa cấu hình, ví dụ **DB connection, config constants**  
- `src/models/` → các schema MongoDB cho **User, Product, Category, Order**  
- `src/controllers/` → xử lý **business logic**, nhận request từ routes, gọi model, trả response  
- `src/routes/` → khai báo RESTful endpoints  
- `src/middleware/` → **Auth, Role check, Validation, Error handling**  
- `src/utils/` → các helper function dùng chung  
- `app.ts` → thiết lập middleware, routes  
- `server.ts` → chạy server  

---

## 2.2 Giải thích MVC Pattern / MVC Pattern Explanation

**MVC = Model – View – Controller** (trong API RESTful, View là JSON)

1. **Model (M)**  
   - Định nghĩa **structure của dữ liệu**, tương tác với database  
   - Ví dụ: `User`, `Product`, `Order`  

2. **Controller (C)**  
   - Xử lý **business logic**  
   - Nhận request từ route → gọi model → trả response  
   - Ví dụ: tạo user mới, lấy danh sách sản phẩm theo filter  

3. **Route (R)**  
   - Định nghĩa **endpoint RESTful**  
   - Map request URL → controller function  
   - Ví dụ: `GET /api/products` → `productController.getAllProducts()`  

**Flow request / request flow:**

```

Client → Route → Controller → Model → Database → Controller → Response → Client

````

**Giải thích / Explanation:**

- Client gửi HTTP request (GET, POST, PUT, DELETE)  
- Route xác định controller nào xử lý  
- Controller thực hiện logic → gọi Model thao tác DB  
- Model trả dữ liệu cho Controller  
- Controller trả response JSON cho client  

---

## 2.3 Thư mục config / Config Folder

**src/config/db.ts** – kết nối MongoDB:

```ts
import mongoose from "mongoose";
import dotenv from "dotenv";

dotenv.config();

const MONGO_URI: string = process.env.MONGO_URI || "";

export const connectDB = async () => {
  try {
    await mongoose.connect(MONGO_URI);
    console.log("MongoDB connected");
  } catch (err) {
    console.error("MongoDB connection error:", err);
    process.exit(1);
  }
};
````

**Giải thích / Explanation:**

* `mongoose.connect` → kết nối MongoDB
* `process.env.MONGO_URI` → lấy từ `.env`
* `process.exit(1)` → dừng server nếu lỗi

---

## 2.4 Thư mục middleware / Middleware Folder

**Auth middleware / Kiểm tra token:**

```ts
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";

export interface AuthRequest extends Request {
  user?: any;
}

export const authMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ message: "Unauthorized" });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET || "secret");
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ message: "Invalid token" });
  }
};
```

**Giải thích / Explanation:**

* `Authorization header` → Bearer token
* `jwt.verify` → xác thực token hợp lệ
* Nếu hợp lệ → `req.user` chứa payload của token
* Nếu không → trả `401 Unauthorized`

---

## 2.5 Thư mục routes / Routes Folder

Ví dụ **product.routes.ts**:

```ts
import express from "express";
import { getAllProducts, getProductById } from "../controllers/product.controller";

const router = express.Router();

router.get("/", getAllProducts);          // GET /api/products
router.get("/:id", getProductById);      // GET /api/products/:id

export default router;
```

**Giải thích / Explanation:**

* `router.get` → khai báo endpoint GET
* Endpoint `/api/products` gọi `getAllProducts` controller
* Endpoint `/api/products/:id` gọi `getProductById` controller

---

## 2.6 Thư mục utils / Utils Folder

Ví dụ helper function:

```ts
export const calculateDiscount = (price: number, salePrice?: number) => {
  if (!salePrice) return 0;
  return Math.round(((price - salePrice) / price) * 100);
};
```

**Giải thích / Explanation:**

* Tính **% giảm giá**
* Nếu không có salePrice → trả 0

---

## ✅ Kết luận Chương 2 / Conclusion Chapter 2

Sau Chương 2, bạn đã:

1. Hiểu **MVC pattern** cho RESTful API
2. Có **cấu trúc thư mục chuẩn**
3. Cài đặt **config**, **middleware cơ bản**, **utils**
4. Đã có ví dụ **route mapping → controller**
5. Sẵn sàng bước tiếp: **CHƯƠNG 3 – Models chi tiết cho User, Product, Category, Order**, với **tất cả field thực tế + JSON mẫu + TypeScript types**.

```

```
