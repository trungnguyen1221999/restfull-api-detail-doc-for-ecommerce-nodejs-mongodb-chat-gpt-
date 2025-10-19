
---

# CHƯƠNG 6 – CATEGORY MODULE / COMPLETE CATEGORY MODULE

## 6.1 Model / src/models/category.model.ts

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface ICategory extends Document {
  name: string;          // Tên danh mục / category name
  description?: string;   // Mô tả / description
  image?: string;         // Hình ảnh / image URL
  createdAt: Date;
  updatedAt: Date;
}

const CategorySchema: Schema = new Schema(
  {
    name: { type: String, required: true, unique: true },
    description: { type: String },
    image: { type: String }
  },
  { timestamps: true }
);

export default mongoose.model<ICategory>("Category", CategorySchema);
```

**Giải thích:**

* `name` unique để tránh trùng tên danh mục
* `image` để hiển thị icon/danh mục giống FPT Shop
* `timestamps` tự động tạo `createdAt` và `updatedAt`

---

## 6.2 Controller / src/controllers/category.controller.ts

```ts
import { Request, Response } from "express";
import Category from "../models/category.model";
import Product from "../models/product.model";

// Tạo danh mục (Admin)
export const createCategory = async (req: Request, res: Response) => {
  try {
    const { name, description, image } = req.body;
    const category = new Category({ name, description, image });
    await category.save();
    return res.status(201).json(category);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Cập nhật danh mục (Admin)
export const updateCategory = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const updateData = req.body;
    const category = await Category.findByIdAndUpdate(id, updateData, { new: true });
    if (!category) return res.status(404).json({ message: "Category not found" });
    return res.json(category);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Xóa danh mục (Admin)
export const deleteCategory = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    // Optional: Xóa tất cả sản phẩm thuộc danh mục này hoặc chỉ xóa danh mục
    await Category.findByIdAndDelete(id);
    return res.json({ message: "Category deleted" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Lấy danh sách category / Get all categories
export const getCategories = async (req: Request, res: Response) => {
  try {
    const categories = await Category.find();
    return res.json(categories);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Lấy sản phẩm theo category + filter nâng cao
export const getProductsByCategory = async (req: Request, res: Response) => {
  try {
    const { id } = req.params; // category ID
    const {
      search, minPrice, maxPrice, rating, topSelling, sortBy, page = 1, limit = 10
    } = req.query;

    const query: any = { category: id };

    // Search theo tên sản phẩm
    if (search) query.name = { $regex: search, $options: "i" };

    // Price range
    if (minPrice || maxPrice) {
      query.price = {};
      if (minPrice) query.price.$gte = Number(minPrice);
      if (maxPrice) query.price.$lte = Number(maxPrice);
    }

    // Rating
    if (rating) query.rating = { $gte: Number(rating) };

    // Top-selling
    if (topSelling === "true") query.topSelling = true;

    // Sort
    let sortOption: any = {};
    switch (sortBy) {
      case "priceAsc": sortOption.price = 1; break;
      case "priceDesc": sortOption.price = -1; break;
      case "rating": sortOption.rating = -1; break;
      case "newest": sortOption.createdAt = -1; break;
      default: sortOption.createdAt = -1;
    }

    const skip = (Number(page) - 1) * Number(limit);

    const products = await Product.find(query)
      .sort(sortOption)
      .skip(skip)
      .limit(Number(limit))
      .populate("category");

    return res.json(products);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};
```

**Giải thích chi tiết / Line-by-line explanation**

* `query: any = { category: id }` → Lấy tất cả sản phẩm trong category đó
* `if (search)` → kết hợp search theo tên sản phẩm + category
* `if (minPrice || maxPrice)` → filter theo khoảng giá
* `if (rating)` → filter theo rating >= giá trị
* `if (topSelling)` → filter top-selling
* `sortOption` → hỗ trợ sort theo priceAsc, priceDesc, rating, newest
* `skip` → tính pagination
* `populate("category")` → trả về thông tin category đầy đủ

> Với controller này, bạn có thể **kết hợp tất cả filter nâng cao như Product module**, nhưng **bắt buộc chỉ trong một category cụ thể**.

---

## 6.3 Routes / src/routes/category.route.ts

```ts
import express from "express";
import {
  createCategory,
  updateCategory,
  deleteCategory,
  getCategories,
  getProductsByCategory
} from "../controllers/category.controller";
import { authMiddleware, adminMiddleware } from "../middlewares/auth.middleware";

const router = express.Router();

// Admin create category / Tạo danh mục (Admin)
router.post("/", authMiddleware, adminMiddleware, createCategory);

// Admin update category / Cập nhật danh mục (Admin)
router.put("/:id", authMiddleware, adminMiddleware, updateCategory);

// Admin delete category / Xóa danh mục (Admin)
router.delete("/:id", authMiddleware, adminMiddleware, deleteCategory);

// Get all categories / Lấy tất cả danh mục
router.get("/", getCategories);

// Get products by category with advanced filter / Lấy sản phẩm theo danh mục + filter nâng cao
router.get("/:id/products", getProductsByCategory);

export default router;
```

---

### Ví dụ request filter nâng cao theo category / Example

```http
GET /api/categories/64a8f.../products?search=iphone&minPrice=500&maxPrice=1500&rating=4&topSelling=true&sortBy=priceAsc&page=1&limit=20
```

* **Category ID:** 64a8f...
* **Search:** iphone
* **Price:** 500 – 1500
* **Rating >= 4**
* **Top-selling**
* **Sort:** priceAsc
* **Page 1, limit 20 sản phẩm**

✅ Kết hợp tất cả filter nâng cao + pagination + sort, chỉ trong category đó, giống thực tế FPT Shop.

---

Bây giờ bạn đã có **Category module + Product filter nâng cao hoàn chỉnh**, có thể copy trực tiếp lên GitHub, tích hợp **full API e-commerce chuẩn thực tế**.

---


