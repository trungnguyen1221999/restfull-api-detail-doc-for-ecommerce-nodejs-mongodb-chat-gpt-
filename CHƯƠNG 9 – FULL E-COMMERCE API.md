

# CHƯƠNG 9 – FULL E-COMMERCE API / COMPLETE API SETUP

---

## 9.1 Cấu trúc thư mục / Folder structure

```
src/
  models/
    user.model.ts
    product.model.ts
    category.model.ts
    cart.model.ts
    order.model.ts
  controllers/
    auth.controller.ts
    user.controller.ts
    product.controller.ts
    category.controller.ts
    cart.controller.ts
    order.controller.ts
  routes/
    auth.route.ts
    user.route.ts
    product.route.ts
    category.route.ts
    cart.route.ts
    order.route.ts
  middlewares/
    auth.middleware.ts
  seed/
    seedData.ts
  app.ts
.env
package.json
```

**Giải thích:**

* `models/` → Mô tả schema MongoDB
* `controllers/` → Logic xử lý request
* `routes/` → RESTful API endpoints
* `middlewares/` → Auth, Admin check
* `seed/` → Seed data mẫu
* `app.ts` → Khởi tạo Express app

---

## 9.2 app.ts – Express setup

```ts
import express from "express";
import mongoose from "mongoose";
import dotenv from "dotenv";
import cors from "cors";

// Routes
import authRoutes from "./routes/auth.route";
import userRoutes from "./routes/user.route";
import productRoutes from "./routes/product.route";
import categoryRoutes from "./routes/category.route";
import cartRoutes from "./routes/cart.route";
import orderRoutes from "./routes/order.route";

dotenv.config();

const app = express();

// Middlewares
app.use(cors());
app.use(express.json());

// Routes
app.use("/api/auth", authRoutes);
app.use("/api/users", userRoutes);
app.use("/api/products", productRoutes);
app.use("/api/categories", categoryRoutes);
app.use("/api/cart", cartRoutes);
app.use("/api/orders", orderRoutes);

// Connect MongoDB
mongoose.connect(process.env.MONGO_URI!, { })
  .then(() => console.log("MongoDB connected"))
  .catch(err => console.error(err));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

**Giải thích:**

* `express.json()` → parse JSON request
* `cors()` → cho phép request từ frontend
* `app.use("/api/xxx")` → mount routes từng module
* `mongoose.connect()` → kết nối DB MongoDB

---

## 9.3 Seed data mẫu / src/seed/seedData.ts

```ts
import mongoose from "mongoose";
import dotenv from "dotenv";
import User from "../models/user.model";
import Product from "../models/product.model";
import Category from "../models/category.model";

dotenv.config();

const seed = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI!);

    // Clear existing data
    await User.deleteMany({});
    await Product.deleteMany({});
    await Category.deleteMany({});

    // Seed users
    const admin = new User({ username: "Admin", email: "admin@fpt.com", password: "123456", role: "admin", isVerified: true });
    const user = new User({ username: "User", email: "user@fpt.com", password: "123456", role: "user", isVerified: true });
    await admin.save();
    await user.save();

    // Seed categories
    const cat1 = new Category({ name: "Laptop", image: "laptop.png" });
    const cat2 = new Category({ name: "Smartphone", image: "phone.png" });
    await cat1.save();
    await cat2.save();

    // Seed products
    const prod1 = new Product({ name: "Laptop Dell XPS", price: 1500, stock: 10, category: cat1._id, topSelling: true });
    const prod2 = new Product({ name: "iPhone 14", price: 1200, stock: 15, category: cat2._id, topSelling: true });
    await prod1.save();
    await prod2.save();

    console.log("Seed data created");
    process.exit();
  } catch (err) {
    console.error(err);
    process.exit(1);
  }
};

seed();
```

---

## 9.4 Swagger API Documentation (Optional)

```ts
// Install: npm install swagger-ui-express swagger-jsdoc
import swaggerUi from "swagger-ui-express";
import swaggerJsdoc from "swagger-jsdoc";

const options = {
  definition: {
    openapi: "3.0.0",
    info: { title: "E-commerce API", version: "1.0.0" }
  },
  apis: ["./src/routes/*.ts"]
};

const specs = swaggerJsdoc(options);
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(specs));
```

* Truy cập `http://localhost:5000/api-docs` để xem tất cả API endpoints
* Bao gồm Auth, User, Product, Category, Cart, Order

---

## 9.5 Chạy server

```bash
npm install
npm run seed      # Seed data mẫu
npm run dev       # Chạy server (ts-node hoặc nodemon)
```

* Server sẽ chạy trên port 5000
* MongoDB kết nối từ `process.env.MONGO_URI`

---

## 9.6 Tổng kết

Bây giờ bạn đã có **full backend API e-commerce chuẩn thực tế**, bao gồm:

1. **Auth Module** (register, login, Google OAuth, verify email, password reset)
2. **User Module** (CRUD, update role, password change)
3. **Product Module** (CRUD, advanced filter, search, pagination, sort, top-selling, variants, images)
4. **Category Module** (CRUD, get products by category + filter)
5. **Cart Module** (add/remove/update items, calculate total)
6. **Order Module** (create order từ cart, order status, filter advanced)
7. **Seed data** + Swagger docs

