
* Model Product
* Controller (CRUD + Advanced Filter/Search/Sort/Pagination)
* Routes
* Giải thích line-by-line **song ngữ**
* Mọi filter kết hợp thực tế giống FPT Shop

---

# CHƯƠNG 5 – PRODUCT MODULE HOÀN CHỈNH / COMPLETE PRODUCT MODULE

## 5.1 Model / src/models/product.model.ts

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IProduct extends Document {
  name: string;                  // Tên sản phẩm / product name
  summary?: string;              // Tóm tắt / summary
  description?: string;          // Mô tả chi tiết / detailed description
  images: string[];              // Mảng hình ảnh / image URLs
  variants?: string[];           // Phiên bản / variants
  price: number;                 // Giá gốc / price
  salePrice?: number;            // Giá khuyến mãi / sale price
  stock: number;                 // Số lượng tồn kho / stock
  rating?: number;               // Đánh giá trung bình / average rating
  category: mongoose.Schema.Types.ObjectId; // Reference category / category ID
  topSelling?: boolean;          // Flag bán chạy / top-selling flag
  createdAt: Date;
  updatedAt: Date;
}

const ProductSchema: Schema = new Schema(
  {
    name: { type: String, required: true },
    summary: { type: String },
    description: { type: String },
    images: { type: [String], default: [] },
    variants: { type: [String], default: [] },
    price: { type: Number, required: true },
    salePrice: { type: Number },
    stock: { type: Number, default: 0 },
    rating: { type: Number, default: 0 },
    category: { type: mongoose.Schema.Types.ObjectId, ref: "Category", required: true },
    topSelling: { type: Boolean, default: false }
  },
  { timestamps: true }
);

export default mongoose.model<IProduct>("Product", ProductSchema);
```

---

## 5.2 Controller / src/controllers/product.controller.ts

```ts
import { Request, Response } from "express";
import Product from "../models/product.model";

// Create product / Tạo sản phẩm (Admin)
export const createProduct = async (req: Request, res: Response) => {
  try {
    const { name, summary, description, images, variants, price, salePrice, stock, category, topSelling } = req.body;
    const product = new Product({ name, summary, description, images, variants, price, salePrice, stock, category, topSelling });
    await product.save();
    return res.status(201).json(product);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Update product / Cập nhật sản phẩm (Admin)
export const updateProduct = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const updateData = req.body;
    const product = await Product.findByIdAndUpdate(id, updateData, { new: true });
    if (!product) return res.status(404).json({ message: "Product not found" });
    return res.json(product);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Delete product / Xóa sản phẩm (Admin)
export const deleteProduct = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    await Product.findByIdAndDelete(id);
    return res.json({ message: "Product deleted" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Get product by ID / Lấy sản phẩm theo ID
export const getProductById = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const product = await Product.findById(id).populate("category");
    if (!product) return res.status(404).json({ message: "Product not found" });
    return res.json(product);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Advanced filter/search/sort/pagination / Filter nâng cao
export const getProductsAdvanced = async (req: Request, res: Response) => {
  try {
    const {
      search,       // Tìm kiếm theo tên / search by name
      category,     // ID danh mục / category id
      minPrice,     // Giá tối thiểu / minimum price
      maxPrice,     // Giá tối đa / maximum price
      rating,       // Rating tối thiểu / minimum rating
      topSelling,   // Sản phẩm bán chạy / top-selling flag
      sortBy,       // Sắp xếp / sorting option
      page = 1,     // Trang / page
      limit = 10    // Số lượng / limit
    } = req.query;

    const query: any = {};

    // 1. Search theo tên / search by name
    if (search) query.name = { $regex: search, $options: "i" };

    // 2. Filter theo category / filter by category
    if (category) query.category = category;

    // 3. Filter theo price range / price range filter
    if (minPrice || maxPrice) {
      query.price = {};
      if (minPrice) query.price.$gte = Number(minPrice);
      if (maxPrice) query.price.$lte = Number(maxPrice);
    }

    // 4. Filter theo rating / rating filter
    if (rating) query.rating = { $gte: Number(rating) };

    // 5. Filter top-selling / top-selling filter
    if (topSelling === "true") query.topSelling = true;

    // 6. Sort / Sắp xếp
    let sortOption: any = {};
    switch (sortBy) {
      case "priceAsc": sortOption.price = 1; break;
      case "priceDesc": sortOption.price = -1; break;
      case "rating": sortOption.rating = -1; break;
      case "newest": sortOption.createdAt = -1; break;
      default: sortOption.createdAt = -1;
    }

    // 7. Pagination / Phân trang
    const skip = (Number(page) - 1) * Number(limit);

    // 8. Execute query / Thực thi truy vấn
    const products = await Product.find(query)
      .sort(sortOption)
      .skip(skip)
      .limit(Number(limit))
      .populate("category");

    return res.json(products);
  } catch (err) {
    console.error(err);
    return res.status(500).json({ message: "Server error" });
  }
};
```

---

### Giải thích filter nâng cao / Advanced filter explanation

* **Tất cả filter kết hợp**: MongoDB tự động `AND` tất cả các điều kiện trong `query`.
* **Category + rating** → `query.category && query.rating`
* **Category + topSelling** → `query.category && query.topSelling`
* **Category + min/max price** → `query.category && query.price`
* **Search + min/max price** → `query.name && query.price`
* **Search + rating** → `query.name && query.rating`
* **Sort + pagination** → áp dụng sau khi filter
* **Populate category** → trả về thông tin đầy đủ category thay vì chỉ ID

---

## 5.3 Routes / src/routes/product.route.ts

```ts
import express from "express";
import {
  createProduct,
  updateProduct,
  deleteProduct,
  getProductById,
  getProductsAdvanced
} from "../controllers/product.controller";
import { authMiddleware, adminMiddleware } from "../middlewares/auth.middleware";

const router = express.Router();

// Admin create product / Tạo sản phẩm (Admin)
router.post("/", authMiddleware, adminMiddleware, createProduct);

// Admin update product / Cập nhật sản phẩm (Admin)
router.put("/:id", authMiddleware, adminMiddleware, updateProduct);

// Admin delete product / Xóa sản phẩm (Admin)
router.delete("/:id", authMiddleware, adminMiddleware, deleteProduct);

// Get product by ID / Lấy sản phẩm theo ID
router.get("/:id", getProductById);

// Get products with advanced filter / Lấy danh sách sản phẩm với filter nâng cao
router.get("/", getProductsAdvanced);

export default router;
```

---

### Ví dụ request filter nâng cao / Example queries

```http
GET /api/products?search=iphone&category=64a8f...&minPrice=500&maxPrice=1500&rating=4&topSelling=true&sortBy=priceAsc&page=2&limit=20
```

* Search: iphone
* Category: 64a8f...
* Price: 500 – 1500
* Rating >= 4
* Top-selling = true
* Sort theo priceAsc
* Page 2, limit 20 sản phẩm

✅ Kết hợp mọi filter nâng cao, pagination và sort, sát thực tế FPT Shop.

