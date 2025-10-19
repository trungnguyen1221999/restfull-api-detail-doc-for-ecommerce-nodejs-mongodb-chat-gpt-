

---

# USER MODULE – COMPLETE / MODULE NGƯỜI DÙNG HOÀN CHỈNH

## 1. Model – Mô hình User (src/models/user.model.ts)

```ts
import mongoose, { Schema, Document } from "mongoose";

export interface IUser extends Document {
  username: string;                 // Tên đăng nhập / username
  email: string;                    // Email
  password: string;                 // Mật khẩu đã hash / hashed password
  avatar?: string;                  // URL ảnh đại diện / avatar URL
  dateOfBirth?: Date;               // Ngày sinh / date of birth
  phoneNumber?: string;             // Số điện thoại / phone number
  role: "admin" | "customer";       // Vai trò / Role
  isVerified: boolean;              // Email đã xác thực / Email verified
  resetPasswordToken?: string;      // Token reset password
  googleId?: string;                // ID Google OAuth / Google OAuth ID
  createdAt: Date;                  // Ngày tạo / created at
  updatedAt: Date;                  // Ngày cập nhật / updated at
}

const UserSchema: Schema = new Schema(
  {
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String },
    avatar: { type: String },
    dateOfBirth: { type: Date },
    phoneNumber: { type: String },
    role: { type: String, enum: ["admin", "customer"], default: "customer" },
    isVerified: { type: Boolean, default: false },
    resetPasswordToken: { type: String },
    googleId: { type: String } // Nếu login bằng Google
  },
  { timestamps: true }
);

export default mongoose.model<IUser>("User", UserSchema);
```

**Giải thích:**

* `password` optional vì user có thể login bằng Google OAuth.
* `googleId` để lưu ID Google nếu đăng nhập OAuth.
* `isVerified` xác nhận email.
* `resetPasswordToken` để reset password.

---

## 2. Middleware – Auth / src/middlewares/auth.middleware.ts

```ts
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";
import User from "../models/user.model";

export interface AuthRequest extends Request {
  user?: any;
}

// Middleware xác thực JWT
export const authMiddleware = async (req: AuthRequest, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith("Bearer "))
    return res.status(401).json({ message: "Unauthorized" });

  const token = authHeader.split(" ")[1];
  try {
    const decoded: any = jwt.verify(token, process.env.JWT_SECRET || "secret");
    const user = await User.findById(decoded.id);
    if (!user) return res.status(401).json({ message: "Unauthorized" });
    req.user = user;
    next();
  } catch (err) {
    return res.status(401).json({ message: "Invalid token" });
  }
};

// Middleware kiểm tra role admin
export const adminMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
  if (!req.user || req.user.role !== "admin")
    return res.status(403).json({ message: "Forbidden" });
  next();
};
```

**Giải thích:**

* `authMiddleware` kiểm tra JWT token, attach user vào `req.user`.
* `adminMiddleware` kiểm tra role admin trước khi cho phép action.

---

## 3. Controller – User / src/controllers/user.controller.ts

```ts
import { Request, Response } from "express";
import User from "../models/user.model";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import crypto from "crypto";
import nodemailer from "nodemailer";

// Đăng ký user + gửi email xác thực
export const registerUser = async (req: Request, res: Response) => {
  try {
    const { username, email, password } = req.body;
    const existingUser = await User.findOne({ email });
    if (existingUser) return res.status(400).json({ message: "Email already exists" });

    const hashedPassword = await bcrypt.hash(password, 10);

    const user = new User({ username, email, password: hashedPassword, role: "customer" });
    await user.save();

    // Email verification token
    const emailToken = jwt.sign({ id: user._id }, process.env.JWT_SECRET || "secret", { expiresIn: "1d" });
    const verificationUrl = `https://yourdomain.com/verify-email?token=${emailToken}`;

    const transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: Number(process.env.SMTP_PORT),
      auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS }
    });

    await transporter.sendMail({
      from: `"Ecom" <${process.env.SMTP_USER}>`,
      to: email,
      subject: "Verify your email",
      html: `<p>Click <a href="${verificationUrl}">here</a> to verify your email.</p>`
    });

    return res.status(201).json({ message: "User created. Please verify your email." });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Xác thực email
export const verifyEmail = async (req: Request, res: Response) => {
  try {
    const { token } = req.query;
    if (!token) return res.status(400).json({ message: "Token missing" });

    const decoded: any = jwt.verify(token as string, process.env.JWT_SECRET || "secret");
    const user = await User.findById(decoded.id);
    if (!user) return res.status(404).json({ message: "User not found" });

    user.isVerified = true;
    await user.save();
    return res.json({ message: "Email verified successfully" });
  } catch (err) {
    return res.status(400).json({ message: "Invalid or expired token" });
  }
};

// Login email/password
export const loginUser = async (req: Request, res: Response) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(400).json({ message: "User not found" });
    if (!user.isVerified) return res.status(400).json({ message: "Email not verified" });

    if (!user.password) return res.status(400).json({ message: "Use Google login" });

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ message: "Invalid credentials" });

    const token = jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET || "secret", { expiresIn: "7d" });

    return res.json({ user, token });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Google OAuth login
export const googleLogin = async (req: Request, res: Response) => {
  try {
    const { googleId, email, username, avatar } = req.body;

    let user = await User.findOne({ googleId });
    if (!user) {
      user = new User({ googleId, email, username, avatar, role: "customer", isVerified: true });
      await user.save();
    }

    const token = jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET || "secret", { expiresIn: "7d" });
    return res.json({ user, token });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Quên mật khẩu
export const forgotPassword = async (req: Request, res: Response) => {
  try {
    const { email } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(404).json({ message: "User not found" });

    const resetToken = crypto.randomBytes(32).toString("hex");
    user.resetPasswordToken = resetToken;
    await user.save();

    const resetUrl = `https://yourdomain.com/reset-password?token=${resetToken}`;

    const transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: Number(process.env.SMTP_PORT),
      auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS }
    });

    await transporter.sendMail({
      from: `"Ecom" <${process.env.SMTP_USER}>`,
      to: email,
      subject: "Reset Password",
      html: `<p>Click <a href="${resetUrl}">here</a> to reset your password.</p>`
    });

    return res.json({ message: "Reset email sent" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Reset password
export const resetPassword = async (req: Request, res: Response) => {
  try {
    const { token, newPassword } = req.body;
    const user = await User.findOne({ resetPasswordToken: token });
    if (!user) return res.status(400).json({ message: "Invalid token" });

    user.password = await bcrypt.hash(newPassword, 10);
    user.resetPasswordToken = undefined;
    await user.save();

    return res.json({ message: "Password reset successful" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Admin update role
export const updateUserRole = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { role } = req.body;
    const user = await User.findById(id);
    if (!user) return res.status(404).json({ message: "User not found" });

    user.role = role;
    await user.save();
    return res.json({ message: "Role updated", user });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// Admin delete user
export const deleteUser = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    await User.findByIdAndDelete(id);
    return res.json({ message: "User deleted" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};

// User update password
export const updatePassword = async (req: Request, res: Response) => {
  try {
    const user = req.user;
    const { oldPassword, newPassword } = req.body;
    if (!user) return res.status(401).json({ message: "Unauthorized" });

    if (!user.password) return res.status(400).json({ message: "Cannot change Google password" });

    const isMatch = await bcrypt.compare(oldPassword, user.password);
    if (!isMatch) return res.status(400).json({ message: "Old password incorrect" });

    user.password = await bcrypt.hash(newPassword, 10);
    await user.save();

    return res.json({ message: "Password updated successfully" });
  } catch (err) {
    return res.status(500).json({ message: "Server error" });
  }
};
```

---

## 4. Routes – User / src/routes/user.route.ts

```ts
import express from "express";
import {
  registerUser,
  verifyEmail,
  loginUser,
  googleLogin,
  forgotPassword,
  resetPassword,
  updateUserRole,
  deleteUser,
  updatePassword
} from "../controllers/user.controller";

import { authMiddleware, adminMiddleware } from "../middlewares/auth.middleware";

const router = express.Router();

// Đăng ký
router.post("/register", registerUser);
// Xác thực email
router.get("/verify-email", verifyEmail);
// Login email/password
router.post("/login", loginUser);
// Login Google OAuth
router.post("/google-login", googleLogin);
// Quên mật khẩu
router.post("/forgot-password", forgotPassword);
// Reset password
router.post("/reset-password", resetPassword);
// Admin update role
router.put("/role/:id", authMiddleware, adminMiddleware, updateUserRole);
// Admin delete user
router.delete("/:id", authMiddleware, adminMiddleware, deleteUser);
// User update password
router.put("/update-password", authMiddleware, updatePassword);

export default router;
```

---

✅ **Module User hoàn chỉnh bao gồm:**

* **Model:** `User` với các trường thực tế (avatar, dob, phone, googleId, isVerified, resetToken)
* **Controller:** Register, Verify email, Login, Google OAuth, Forgot/Reset password, Update role/password, Delete user
* **Route:** đầy đủ các endpoint RESTful, role-based authorization
* **Middleware:** JWT authentication + admin check
* **Song ngữ + line-by-line explanation**
\
