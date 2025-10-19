

# CHƯƠNG 7 – CART & ORDER MODULE / COMPLETE CART & ORDER MODULE

---

## 7.1 Cart Module

### 7.1.1 Model / src/models/cart.model.ts

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface ICartItem {
  product: mongoose.Schema.Types.ObjectId; // Product ID
  quantity: number;                        // Số lượng / quantity
}

export interface ICart extends Document {
  user: mongoose.Schema.Types.ObjectId;    // User ID
  items: ICartItem[];                      // Mảng sản phẩm / cart items
  totalPrice: number;                      // Tổng giá / total price
  createdAt: Date;
  updatedAt: Date;
}

const CartItemSchema: Schema = new Schema({
  product: { type: mongoose.Schema.Types.ObjectId, ref: "Product", required: true },
  quantity: { type: Number, required: true, min: 1 }
});

const CartSchema: Schema = new Schema(
  {
    user: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true, unique: true },
    items: { type: [CartItemSchema], default: [] },
    totalPrice: { type: Number, default: 0 }
  },
  { timestamps: true }
);

export default mongoose.model<ICart>("Cart", CartSchema);
```

**Giải thích:**

* Một user chỉ có 1 cart duy nhất (`unique: true`)
* `items` chứa product ID + quantity
* `totalPrice` lưu tổng tiền, tính động khi thêm/xóa/update sản phẩm

---

### 7.1.2 Controller / src/controllers/cart.controller.ts

```ts
import { Request, Response } from "express";
import Cart from "../models/cart.model";
import Product from "../models/product.model";

// Helper tính tổng tiền / calculate total price
const calculateTotalPrice = async (items: { product: string, quantity: number }[]) => {
  let total = 0;
  for (const item of items) {
    const product = await Product.findById(item.product);
    if (product) {
      total += (product.salePrice || product.price) * item.quantity;
    }
  }
  return total;
};

// Thêm sản phẩm vào cart / add product
export const addToCart = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id; // user từ auth middleware
    const { productId, quantity } = req.body;

    let cart = await Cart.findOne({ user: userId });
    if (!cart) cart = new Cart({ user: userId, items: [], totalPrice: 0 });

    const existingItem = cart.items.find(i => i.product.toString() === productId);
    if (existingItem) existingItem.quantity += quantity;
    else cart.items.push({ product: productId, quantity });

    cart.totalPrice = await calculateTotalPrice(cart.items);
    await cart.save();

    return res.json(cart);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Xóa sản phẩm khỏi cart / remove product
export const removeFromCart = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id;
    const { productId } = req.params;

    const cart = await Cart.findOne({ user: userId });
    if (!cart) return res.status(404).json({ message: "Cart not found" });

    cart.items = cart.items.filter(i => i.product.toString() !== productId);
    cart.totalPrice = await calculateTotalPrice(cart.items);
    await cart.save();

    return res.json(cart);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Cập nhật số lượng sản phẩm / update quantity
export const updateCartItem = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id;
    const { productId, quantity } = req.body;

    const cart = await Cart.findOne({ user: userId });
    if (!cart) return res.status(404).json({ message: "Cart not found" });

    const item = cart.items.find(i => i.product.toString() === productId);
    if (!item) return res.status(404).json({ message: "Product not in cart" });

    item.quantity = quantity;
    cart.totalPrice = await calculateTotalPrice(cart.items);
    await cart.save();

    return res.json(cart);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Lấy cart của user / get user's cart
export const getCart = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id;
    const cart = await Cart.findOne({ user: userId }).populate("items.product");
    if (!cart) return res.status(404).json({ message: "Cart not found" });
    return res.json(cart);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};
```

---

### 7.1.3 Routes / src/routes/cart.route.ts

```ts
import express from "express";
import { authMiddleware } from "../middlewares/auth.middleware";
import { addToCart, removeFromCart, updateCartItem, getCart } from "../controllers/cart.controller";

const router = express.Router();

router.use(authMiddleware);

// Add product to cart
router.post("/add", addToCart);
// Remove product
router.delete("/remove/:productId", removeFromCart);
// Update quantity
router.put("/update", updateCartItem);
// Get cart
router.get("/", getCart);

export default router;
```

---

## 7.2 Order Module

### 7.2.1 Model / src/models/order.model.ts

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IOrderItem {
  product: mongoose.Schema.Types.ObjectId;
  quantity: number;
  price: number; // giá tại thời điểm đặt / price at order time
}

export interface IOrder extends Document {
  user: mongoose.Schema.Types.ObjectId;
  items: IOrderItem[];
  totalPrice: number;
  status: "pending" | "processing" | "on delivery" | "delivered" | "payment declined" | "completed";
  paymentMethod: string;
  createdAt: Date;
  updatedAt: Date;
}

const OrderItemSchema: Schema = new Schema({
  product: { type: mongoose.Schema.Types.ObjectId, ref: "Product", required: true },
  quantity: { type: Number, required: true },
  price: { type: Number, required: true }
});

const OrderSchema: Schema = new Schema(
  {
    user: { type: mongoose.Schema.Types.ObjectId, ref: "User", required: true },
    items: { type: [OrderItemSchema], required: true },
    totalPrice: { type: Number, required: true },
    status: { type: String, default: "pending", enum: ["pending", "processing", "on delivery", "delivered", "payment declined", "completed"] },
    paymentMethod: { type: String, required: true }
  },
  { timestamps: true }
);

export default mongoose.model<IOrder>("Order", OrderSchema);
```

---

### 7.2.2 Controller / src/controllers/order.controller.ts

```ts
import { Request, Response } from "express";
import Order from "../models/order.model";
import Cart from "../models/cart.model";

// Tạo order từ cart / create order
export const createOrder = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id;
    const { paymentMethod } = req.body;

    const cart = await Cart.findOne({ user: userId }).populate("items.product");
    if (!cart || cart.items.length === 0) return res.status(400).json({ message: "Cart is empty" });

    const orderItems = cart.items.map(i => ({
      product: i.product._id,
      quantity: i.quantity,
      price: i.product.salePrice || i.product.price
    }));

    const totalPrice = orderItems.reduce((acc, item) => acc + item.price * item.quantity, 0);

    const order = new Order({ user: userId, items: orderItems, totalPrice, paymentMethod, status: "pending" });
    await order.save();

    // Xóa cart sau khi đặt hàng / clear cart
    cart.items = [];
    cart.totalPrice = 0;
    await cart.save();

    return res.status(201).json(order);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Cập nhật status order / update order status (Admin)
export const updateOrderStatus = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { status } = req.body;

    const order = await Order.findById(id);
    if (!order) return res.status(404).json({ message: "Order not found" });

    order.status = status;
    await order.save();

    return res.json(order);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Lấy orders của user / get user's orders
export const getUserOrders = async (req: Request, res: Response) => {
  try {
    const userId = req.user._id;
    const orders = await Order.find({ user: userId }).populate("items.product");
    return res.json(orders);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Lấy tất cả orders (Admin) với filter nâng cao
export const getOrdersAdvanced = async (req: Request, res: Response) => {
  try {
    const { status, userId, startDate, endDate, page = 1, limit = 20 } = req.query;

    const query: any = {};

    if (status) query.status = status;
    if (userId) query.user = userId;
    if (startDate || endDate) query.createdAt = {};
    if (startDate) query.createdAt.$gte = new Date(startDate as string);
    if (endDate) query.createdAt.$lte = new Date(endDate as string);

    const skip = (Number(page) - 1) * Number(limit);

    const orders = await Order.find(query)
      .sort({ createdAt: -1 })
      .skip(skip)
      .limit(Number(limit))
      .populate("items.product")
      .populate("user");

    return res.json(orders);
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};
```

---

### 7.2.3 Routes / src/routes/order.route.ts

```ts
import express from "express";
import { authMiddleware, adminMiddleware } from "../middlewares/auth.middleware";
import {
  createOrder,
  updateOrderStatus,
  getUserOrders,
  getOrdersAdvanced
} from "../controllers/order.controller";

const router = express.Router();

// Auth middleware áp dụng cho tất cả routes
router.use(authMiddleware);

// Tạo order / Create order
router.post("/", createOrder);

// Lấy orders của user / Get user orders
router.get("/my-orders", getUserOrders);

// Admin cập nhật order status / Update order status
router.put("/:id/status", adminMiddleware, updateOrderStatus);

// Admin lấy tất cả orders với filter nâng cao
router.get("/", adminMiddleware, getOrdersAdvanced);

export default router;
```

---

### Ví dụ request filter order nâng cao / Example

```http
GET /api/orders?status=processing&userId=64b1f...&startDate=2025-01-01&endDate=2025-01-31&page=1&limit=20
```

* Filter theo `status=processing`
* Theo userId
* Theo khoảng thời gian từ 01-01-2025 đến 31-01-2025
* Pagination: page 1, 20 orders per page

✅ Kết hợp tất cả filter nâng cao, pagination, sort, sát thực tế FPT Shop.
