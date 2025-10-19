
````markdown
# CHƯƠNG 3 – MODELS CHI TIẾT (User, Category, Product, Cart, Order) / CHAPTER 3 – DETAILED MODELS

Mục tiêu: Định nghĩa **tất cả Models** cho RESTful API e-commerce theo mô hình **FPT Shop**, bao gồm **User, Category, Product, Cart, Order**, với **mọi thuộc tính thực tế**, **TypeScript interfaces**, **JSON mẫu**, và **giải thích chi tiết từng dòng code**.

---

## 3.1 User Model / Người dùng

**src/models/user.model.ts**

```ts
// Import mongoose để tạo Schema và Document interface
// Import mongoose to define Schema and Document interface
import mongoose, { Schema, Document } from "mongoose";

// Định nghĩa interface TypeScript cho User
// TypeScript interface for User
export interface IUser extends Document {
  username: string;                 // Tên đăng nhập / username
  email: string;                    // Email
  password: string;                 // Mật khẩu đã hash / hashed password
  avatar?: string;                  // URL ảnh đại diện / avatar URL
  dateOfBirth?: Date;               // Ngày sinh / date of birth
  phoneNumber?: string;             // Số điện thoại / phone number
  role: "admin" | "customer";       // Vai trò / Role
  createdAt: Date;                  // Ngày tạo / created at
  updatedAt: Date;                  // Ngày cập nhật / updated at
}

// Tạo Schema User
// Create User Schema
const UserSchema: Schema = new Schema(
  {
    // username: bắt buộc, duy nhất
    username: { type: String, required: true, unique: true },
    // email: bắt buộc, duy nhất
    email: { type: String, required: true, unique: true },
    // password: bắt buộc, sẽ hash trước khi lưu
    password: { type: String, required: true },
    // avatar: optional, URL ảnh đại diện
    avatar: { type: String },
    // dateOfBirth: optional, kiểu Date
    dateOfBirth: { type: Date },
    // phoneNumber: optional
    phoneNumber: { type: String },
    // role: enum ["admin", "customer"], default là "customer"
    role: { type: String, enum: ["admin", "customer"], default: "customer" }
  },
  // timestamps: tự động thêm createdAt và updatedAt
  { timestamps: true }
);

// Export model User
export default mongoose.model<IUser>("User", UserSchema);
````

**Giải thích line-by-line / Line-by-line explanation**

1. `import mongoose, { Schema, Document } from "mongoose";`

   * Nhập `mongoose` để thao tác MongoDB, `Schema` để định nghĩa cấu trúc collection, `Document` để gán type TypeScript.
   * Import `mongoose` for MongoDB operations, `Schema` to define collection structure, `Document` for TypeScript type.

2. `export interface IUser extends Document { ... }`

   * Định nghĩa interface TypeScript, giúp **kiểm tra type khi viết code**.
   * Define TypeScript interface for type checking.

3. `const UserSchema: Schema = new Schema({ ... }, { timestamps: true });`

   * Tạo Schema MongoDB. `timestamps: true` tự động tạo `createdAt` và `updatedAt`.
   * Create MongoDB Schema. `timestamps: true` auto-generates `createdAt` and `updatedAt`.

4. `export default mongoose.model<IUser>("User", UserSchema);`

   * Tạo model User và xuất ra để sử dụng trong Controller.
   * Create User model and export for Controller usage.

---

## 3.2 Category Model / Danh mục sản phẩm

**src/models/category.model.ts**

```ts
import mongoose, { Schema, Document } from "mongoose";

// Interface TypeScript cho Category
export interface ICategory extends Document {
  name: string;          // Tên danh mục / category name
  image?: string;        // Ảnh danh mục / optional category image
  description?: string;  // Mô tả danh mục / optional description
  createdAt: Date;       // Ngày tạo / created at
  updatedAt: Date;       // Ngày cập nhật / updated at
}

// Schema Category
const CategorySchema: Schema = new Schema(
  {
    name: { type: String, required: true, unique: true },  // bắt buộc và duy nhất
    image: { type: String },                                // optional
    description: { type: String }                           // optional
  },
  { timestamps: true } // tự động tạo createdAt, updatedAt
);

export default mongoose.model<ICategory>("Category", CategorySchema);
```

**Giải thích line-by-line / Line-by-line explanation**

* `name: { type: String, required: true, unique: true }`

  * Bắt buộc, không trùng lặp.
  * Required, must be unique.

* `image: { type: String }`

  * URL ảnh đại diện cho category, optional.
  * Optional category image URL.

* `description: { type: String }`

  * Mô tả ngắn, optional.
  * Optional short description.

---

## 3.3 Product Model / Sản phẩm

**src/models/product.model.ts**

```ts
import mongoose, { Schema, Document } from "mongoose";

// Interface TypeScript
export interface IProduct extends Document {
  name: string;
  summary: string;
  detailDescription: string;
  images: string[];
  price: number;
  salePrice?: number;
  stock: number;
  rating?: number;
  variants?: { name: string; options: string[] }[];
  category: mongoose.Types.ObjectId;
  createdAt: Date;
  updatedAt: Date;
}

// Schema Product
const ProductSchema: Schema = new Schema(
  {
    name: { type: String, required: true },                    // Tên sản phẩm / Product name
    summary: { type: String, required: true },                 // Tóm tắt / short summary
    detailDescription: { type: String, required: true },       // Mô tả chi tiết / detailed description
    images: { type: [String], default: [] },                   // Mảng hình ảnh / product images
    price: { type: Number, required: true },                   // Giá gốc / original price
    salePrice: { type: Number },                                // Giá khuyến mãi / sale price
    stock: { type: Number, default: 0 },                        // Số lượng tồn kho / stock
    rating: { type: Number, min: 0, max: 5, default: 0 },      // Rating trung bình / average rating
    variants: { type: [{ name: String, options: [String] }] }, // Variants (màu sắc, dung lượng...) / variants
    category: { type: mongoose.Schema.Types.ObjectId, ref: "Category", required: true } // Tham chiếu Category
  },
  { timestamps: true } // createdAt và updatedAt tự động
);

export default mongoose.model<IProduct>("Product", ProductSchema);
```

**Giải thích line-by-line / Line-by-line explanation**

* `images: { type: [String], default: [] }` → mảng URL ảnh, default rỗng.

  * Array of image URLs, default empty.

* `variants: { type: [{ name: String, options: [String] }] }` → lưu các biến thể sản phẩm, ví dụ: màu sắc, dung lượng.

  * Store product variants, e.g., color, storage.

* `category: { type: mongoose.Schema.Types.ObjectId, ref: "Category", required: true }` → liên kết tới Category.

  * Reference to Category document.

---

## 3.4 Cart Model / Giỏ hàng

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface ICartItem {
  product: mongoose.Types.ObjectId;
  quantity: number;
  selectedVariants?: { name: string; option: string }[];
}

export interface ICart extends Document {
  user: mongoose.Types.ObjectId;
  items: ICartItem[];
  createdAt: Date;
  updatedAt: Date;
}

const CartSchema: Schema = new Schema(
  {
    user: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true }, // Ai sở hữu giỏ hàng
    items: [
      {
        product: { type: mongoose.Schema.Types.ObjectId, ref: "Product", required: true }, // sản phẩm
        quantity: { type: Number, default: 1 },                                           // số lượng
        selectedVariants: [{ name: String, option: String }]                              // variants chọn
      }
    ]
  },
  { timestamps: true }
);

export default mongoose.model<ICart>("Cart", CartSchema);
```

**Giải thích line-by-line / Line-by-line explanation**

* `user: { ... }` → liên kết Cart với User.

  * Connect cart to owner User.

* `items: [...]` → mỗi item gồm product, quantity, variants.

  * Each cart item contains product, quantity, selected variants.

---

## 3.5 Order Model / Đơn hàng

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IOrderItem {
  product: mongoose.Types.ObjectId;
  quantity: number;
  price: number;
  selectedVariants?: { name: string; option: string }[];
}

export interface IOrder extends Document {
  user: mongoose.Types.ObjectId;
  items: IOrderItem[];
  total: number;
  status: "pending" | "processing" | "onDelivery" | "delivered" | "paymentDeclined" | "completed";
  paymentMethod: "cash" | "card" | "paypal";
  shippingAddress: string;
  createdAt: Date;
  updatedAt: Date;
}

const OrderSchema: Schema = new Schema(
  {
    user: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true },
    items: [
      {
        product: { type: mongoose.Schema.Types.ObjectId, ref: "Product", required: true },
        quantity: { type: Number, required: true },
        price: { type: Number, required: true },
        selectedVariants: [{ name: String, option: String }]
      }
    ],
    total: { type: Number, required: true },
    status: {
      type: String,
      enum: ["pending", "processing", "onDelivery", "delivered", "paymentDeclined", "completed"],
      default: "pending"
    },
    paymentMethod: { type: String, enum: ["cash", "card", "paypal"], default: "cash" },
    shippingAddress: { type: String, required: true }
  },
  { timestamps: true }
);

export default mongoose.model<IOrder>("Order", OrderSchema);
```

**Giải thích line-by-line / Line-by-line explanation**

*


`status` → quản lý trạng thái order thực tế: pending → processing → onDelivery → delivered.

* Track real order lifecycle.

* `paymentMethod` → cash, card, paypal.

  * Payment method used.

* `total` → tổng giá = sum(items.price * quantity).

  * Total price of order.

---

✅ **Kết luận Chương 3**

* Tất cả Models **line-by-line explained**
* **Song ngữ Anh–Việt**
* Bao gồm **User, Category, Product, Cart, Order**


Bạn có muốn mình tiếp tục luôn không?
