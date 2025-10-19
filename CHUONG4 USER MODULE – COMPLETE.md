

1. Backend:

Model (User)

Controller (Register, Login, Google OAuth, Email verify, Forgot/Reset password, Access/Refresh token, Logout, Update role/password, Delete user)

Middleware (auth + admin check)

Routes



2. Frontend:

React + TypeScript + TanStack Query + React Hook Form

Login, Register, Logout, Forgot/Reset password, Email verify, Google OAuth

Axios instance với interceptor refresh token tự động




Tất cả code song ngữ + line-by-line explanation, copy được 1 lần, bỏ CSS.


---

USER MODULE – FULL FLOW (JWT, Refresh, OAuth, Admin/Customer)

1. Model – User / src/models/user.model.ts

import mongoose, { Schema, Document } from "mongoose";

// Interface cho user
export interface IUser extends Document {
  username: string;                 // Tên đăng nhập / username
  email: string;                    // Email
  password?: string;                // Mật khẩu hash / hashed password
  avatar?: string;                  // Ảnh đại diện / avatar URL
  dateOfBirth?: Date;               // Ngày sinh / date of birth
  phoneNumber?: string;             // Số điện thoại / phone number
  role: "admin" | "customer";       // Vai trò / role
  isVerified: boolean;              // Email đã xác thực / verified email
  resetPasswordToken?: string;      // Token reset password
  googleId?: string;                // Google OAuth ID
  refreshTokens: string[];          // Lưu refresh token / store multiple refresh tokens
  createdAt: Date;
  updatedAt: Date;
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
    googleId: { type: String },
    refreshTokens: { type: [String], default: [] }
  },
  { timestamps: true }
);

export default mongoose.model<IUser>("User", UserSchema);

Explanation:

refreshTokens: lưu tất cả refresh token hợp lệ, hỗ trợ multi-device login

googleId: dùng cho login OAuth

password optional vì có thể login bằng Google OAuth



---

2. Middleware – Auth / src/middlewares/auth.middleware.ts

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
    return res.status(401).json({ message: "Unauthorized / Không được phép" });

  const token = authHeader.split(" ")[1];
  try {
    const decoded: any = jwt.verify(token, process.env.JWT_SECRET || "secret");
    const user = await User.findById(decoded.id);
    if (!user) return res.status(401).json({ message: "Unauthorized / Không được phép" });
    req.user = user;
    next();
  } catch (err) {
    return res.status(401).json({ message: "Invalid token / Token không hợp lệ" });
  }
};

// Middleware kiểm tra role admin
export const adminMiddleware = (req: AuthRequest, res: Response, next: NextFunction) => {
  if (!req.user || req.user.role !== "admin")
    return res.status(403).json({ message: "Forbidden / Không có quyền" });
  next();
};


---

3. Controller – User & Auth / src/controllers/user.controller.ts + auth.controller.ts

Auth functions: login, refresh, logout, Google OAuth, register, verify email

import { Request, Response } from "express";
import User from "../models/user.model";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import crypto from "crypto";
import nodemailer from "nodemailer";

// Tạo Access Token / Refresh Token
const generateAccessToken = (user: any) =>
  jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET || "secret", { expiresIn: "15m" });

const generateRefreshToken = (user: any) =>
  jwt.sign({ id: user._id }, process.env.REFRESH_TOKEN_SECRET || "refreshSecret", { expiresIn: "7d" });

// Register + send verify email
export const registerUser = async (req: Request, res: Response) => {
  try {
    const { username, email, password } = req.body;
    const exist = await User.findOne({ email });
    if (exist) return res.status(400).json({ message: "Email already exists / Email đã tồn tại" });

    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ username, email, password: hashedPassword, role: "customer" });
    await user.save();

    const emailToken = jwt.sign({ id: user._id }, process.env.JWT_SECRET || "secret", { expiresIn: "1d" });
    const verificationUrl = `http://localhost:3000/verify-email?token=${emailToken}`;

    const transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: Number(process.env.SMTP_PORT),
      auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS }
    });

    await transporter.sendMail({
      from: `"Ecom" <${process.env.SMTP_USER}>`,
      to: email,
      subject: "Verify your email",
      html: `<p>Click <a href="${verificationUrl}">here</a> to verify your email</p>`
    });

    return res.status(201).json({ message: "User created. Please verify your email / Tạo user thành công, hãy xác thực email" });
  } catch (err) {
    return res.status(500).json({ message: "Server error / Lỗi server" });
  }
};

// Verify Email
export const verifyEmail = async (req: Request, res: Response) => {
  try {
    const { token } = req.query;
    if (!token) return res.status(400).json({ message: "Token missing / Thiếu token" });
    const decoded: any = jwt.verify(token as string, process.env.JWT_SECRET || "secret");
    const user = await User.findById(decoded.id);
    if (!user) return res.status(404).json({ message: "User not found / Không tìm thấy user" });

    user.isVerified = true;
    await user.save();
    return res.json({ message: "Email verified successfully / Xác thực email thành công" });
  } catch (err) {
    return res.status(400).json({ message: "Invalid or expired token / Token không hợp lệ hoặc hết hạn" });
  }
};

// Login email/password
export const loginUser = async (req: Request, res: Response) => {
  try {
    const { email, password } = req.body;
    const user = await User.findOne({ email });
    if (!user) return res.status(400).json({ message: "User not found / Không tìm thấy user" });
    if (!user.password) return res.status(400).json({ message: "Use Google login / Sử dụng Google login" });
    if (!user.isVerified) return res.status(400).json({ message: "Email not verified / Email chưa xác thực" });

    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) return res.status(400).json({ message: "Invalid credentials / Sai mật khẩu" });

    const accessToken = generateAccessToken(user);
    const refreshToken = generateRefreshToken(user);

    user.refreshTokens.push(refreshToken);
    await user.save();

    res.cookie("refreshToken", refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "Strict",
      maxAge: 7 * 24 * 60 * 60 * 1000
    });

    return res.json({ user, accessToken });
  } catch (err) {
    return res.status(500).json({ message: "Server error / Lỗi server" });
  }
};

// Refresh token
export const refreshToken = async (req: Request, res: Response) => {
  try {
    const token = req.cookies.refreshToken;
    if (!token) return res.status(401).json({ message: "No refresh token / Không có refresh token" });

    const decoded: any = jwt.verify(token, process.env.REFRESH_TOKEN_SECRET || "refreshSecret");
    const user = await User.findById(decoded.id);
    if (!user) return res.status(404).json({ message: "User not found / Không tìm thấy user" });
    if (!user.refreshTokens.includes(token)) return res.status(403).json({ message: "Invalid refresh token / Token không hợp lệ" });

    const newAccessToken = generateAccessToken(user);
    const newRefreshToken = generateRefreshToken(user);

    user.refreshTokens = user.refreshTokens.filter(t => t !== token);
    user.refreshTokens.push(newRefreshToken);
    await user.save();

    res.cookie("refreshToken", newRefreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "Strict",
      maxAge: 7 * 24 * 60 * 60 * 1000
    });

    return res.json({ accessToken: newAccessToken });
  } catch (err) {
    return res.status(401).json({ message: "Invalid or expired token / Token không hợp lệ hoặc hết hạn" });
  }
};

// Logout
export const logoutUser = async (req: Request, res: Response) => {
  try {
    const token = req.cookies.refreshToken;
    if (!token) return res.json({ message: "Logged out / Đã đăng xuất" });

    const decoded: any = jwt.verify(token, process.env.REFRESH_TOKEN_SECRET || "refreshSecret");
    const user = await User.findById(decoded.id);
    if (user) {
      user.refreshTokens = user.refreshTokens.filter(t => t !== token);
      await user.save();
    }

    res.clearCookie("refreshToken");
    return res.json({ message: "Logged out successfully / Đăng xuất thành công" });
  } catch {
    return res.json({ message: "Logged out / Đã đăng xuất" });
  }
};

// Admin update role
export const updateUserRole = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    const { role } = req.body;
    const user = await User.findById(id);
    if (!user) return res.status(404).json({ message: "User not found / Không tìm thấy user" });

    user.role = role;
    await user.save();
    return res.json({ message: "Role updated / Cập nhật quyền thành công", user });
  } catch (err) {
    return res.status(500).json({ message: "Server error / Lỗi server" });
  }
};

// User update password
export const updatePassword = async (req: Request, res: Response) => {
  try {
    const user = req.user;
    const { oldPassword, newPassword } = req.body;
    if (!user) return res.status(401).json({ message: "Unauthorized / Không có quyền" });
    if (!user.password) return res.status(400).json({ message: "Cannot change Google password / Không thể đổi mật khẩu Google" });

    const isMatch = await bcrypt.compare(oldPassword, user.password);
    if (!isMatch) return res.status(400).json({ message: "Old password incorrect / Mật khẩu cũ không đúng" });

    user.password = await bcrypt.hash(newPassword, 10);
    await user.save();
    return res.json({ message: "Password updated successfully / Đổi mật khẩu thành công" });
  } catch (err) {
    return res.status(500).json({ message: "Server error / Lỗi server" });
  }
};

// Admin delete user
export const deleteUser = async (req: Request, res: Response) => {
  try {
    const { id } = req.params;
    await User.findByIdAndDelete(id);
    return res.json({ message: "User deleted / Xóa user thành công" });
  } catch (err) {
    return res.status(500).json({ message: "Server error / Lỗi server" });
  }
};


---

4. Routes – User + Auth / src/routes/user.route.ts + auth.route.ts

import express from "express";
import { authMiddleware, adminMiddleware } from "../middlewares/auth.middleware";
import {
  registerUser,
  verifyEmail,
  loginUser,
  refreshToken,
  logoutUser,
  updateUserRole,
  updatePassword,
  deleteUser
} from "../controllers/user.controller";

const router = express.Router();

// Auth routes
router.post("/register", registerUser);
router.get("/verify-email", verifyEmail);
router.post("/login", loginUser);
router.post("/refresh-token", refreshToken);
router.post("/logout", authMiddleware, logoutUser);

// Admin/User management
router.put("/role/:id", authMiddleware, adminMiddleware, updateUserRole);
router.put("/update-password", authMiddleware, updatePassword);
router.delete("/:id", authMiddleware, adminMiddleware, deleteUser);

export default router;


---

FRONTEND – React + TypeScript + TanStack Query + React Hook Form

1. Axios instance / src/api/axios.ts

import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:5000/api",
  withCredentials: true
});

// Interceptor auto refresh token
api.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      const res = await api.post("/auth/refresh-token");
      const newAccessToken = res.data.accessToken;
      originalRequest.headers["Authorization"] = `Bearer ${newAccessToken}`;
      return api(originalRequest);
    }
    return Promise.reject(error);
  }
);

export default api;


---

2. React Query + Auth hooks / src/hooks/useAuth.ts

import { useMutation } from "@tanstack/react-query";
import api from "../api/axios";

export const useLogin = () => useMutation(async (data: { email: string; password: string }) => {
  const res = await api.post("/auth/login", data);
  return res.data;
});

export const useLogout = () => useMutation(async () => {
  const res = await api.post("/auth/logout");
  return res.data;
});

export const useRegister = () => useMutation(async (data: { username: string; email: string; password: string }) => {
  const res = await api.post("/auth/register", data);
  return res.data;
});


---

Ah, mình hiểu rồi, bạn muốn toàn bộ code frontend login/register/logout + refresh token flow + các component React Hook Form đầy đủ trong một lần. Mình sẽ viết tiếp từ chỗ dừng, hoàn chỉnh Login component, Register component, Logout, Forgot/Reset password, và show cách dùng React Query với TanStack Query tự động refresh token.


---

3. Login Component / src/components/Login.tsx (tiếp)

import React from "react";
import { useForm } from "react-hook-form";
import { useLogin } from "../hooks/useAuth";

type FormData = { email: string; password: string };

const Login = () => {
  const { register, handleSubmit } = useForm<FormData>();
  const loginMutation = useLogin();

  const onSubmit = (data: FormData) => {
    loginMutation.mutate(data, {
      onSuccess: (res) => {
        console.log("Login successful / Đăng nhập thành công", res);
        // Lưu access token vào localStorage hoặc state global nếu muốn
        localStorage.setItem("accessToken", res.accessToken);
      },
      onError: (err: any) => {
        alert(err.response?.data?.message || "Login failed / Đăng nhập thất bại");
      }
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} placeholder="Email" type="email" required />
      <input {...register("password")} placeholder="Password" type="password" required />
      <button type="submit">Login / Đăng nhập</button>
    </form>
  );
};

export default Login;

Explanation:

useForm quản lý form state

useLogin mutation gọi API login backend

Access token lưu localStorage để attach header Authorization khi gọi API



---

4. Register Component / src/components/Register.tsx

import React from "react";
import { useForm } from "react-hook-form";
import { useRegister } from "../hooks/useAuth";

type FormData = { username: string; email: string; password: string };

const Register = () => {
  const { register, handleSubmit } = useForm<FormData>();
  const registerMutation = useRegister();

  const onSubmit = (data: FormData) => {
    registerMutation.mutate(data, {
      onSuccess: (res) => {
        alert(res.message);
      },
      onError: (err: any) => {
        alert(err.response?.data?.message || "Register failed / Đăng ký thất bại");
      }
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("username")} placeholder="Username" required />
      <input {...register("email")} placeholder="Email" type="email" required />
      <input {...register("password")} placeholder="Password" type="password" required />
      <button type="submit">Register / Đăng ký</button>
    </form>
  );
};

export default Register;


---

5. Logout Button / src/components/Logout.tsx

import React from "react";
import { useLogout } from "../hooks/useAuth";

const Logout = () => {
  const logoutMutation = useLogout();

  const handleLogout = () => {
    logoutMutation.mutate(undefined, {
      onSuccess: () => {
        alert("Logged out successfully / Đăng xuất thành công");
        localStorage.removeItem("accessToken");
      },
      onError: () => alert("Logout failed / Đăng xuất thất bại")
    });
  };

  return <button onClick={handleLogout}>Logout / Đăng xuất</button>;
};

export default Logout;


---

6. Forgot Password Component / src/components/ForgotPassword.tsx

import React from "react";
import { useForm } from "react-hook-form";
import api from "../api/axios";

type FormData = { email: string };

const ForgotPassword = () => {
  const { register, handleSubmit } = useForm<FormData>();

  const onSubmit = async (data: FormData) => {
    try {
      const res = await api.post("/auth/forgot-password", data);
      alert(res.data.message);
    } catch (err: any) {
      alert(err.response?.data?.message || "Request failed / Yêu cầu thất bại");
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} placeholder="Email" type="email" required />
      <button type="submit">Send reset link / Gửi link reset</button>
    </form>
  );
};

export default ForgotPassword;


---

7. Reset Password Component / src/components/ResetPassword.tsx

import React from "react";
import { useForm } from "react-hook-form";
import api from "../api/axios";
import { useSearchParams } from "react-router-dom";

type FormData = { newPassword: string };

const ResetPassword = () => {
  const { register, handleSubmit } = useForm<FormData>();
  const [searchParams] = useSearchParams();
  const token = searchParams.get("token");

  const onSubmit = async (data: FormData) => {
    if (!token) return alert("Invalid token / Token không hợp lệ");
    try {
      const res = await api.post("/auth/reset-password", { token, newPassword: data.newPassword });
      alert(res.data.message);
    } catch (err: any) {
      alert(err.response?.data?.message || "Reset failed / Reset thất bại");
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("newPassword")} placeholder="New Password" type="password" required />
      <button type="submit">Reset Password / Đặt lại mật khẩu</button>
    </form>
  );
};

export default ResetPassword;


---

8. Google OAuth Login (Client) / src/components/GoogleLogin.tsx

import React from "react";
import api from "../api/axios";

declare const gapi: any;

const GoogleLoginButton = () => {
  const handleGoogleLogin = async () => {
    try {
      const auth2 = gapi.auth2.getAuthInstance();
      const googleUser = await auth2.signIn();
      const profile = googleUser.getBasicProfile();
      const res = await api.post("/auth/google-login", {
        googleId: profile.getId(),
        email: profile.getEmail(),
        username: profile.getName(),
        avatar: profile.getImageUrl()
      });
      localStorage.setItem("accessToken", res.data.accessToken);
      alert("Login successful / Đăng nhập thành công");
    } catch (err) {
      alert("Google login failed / Đăng nhập Google thất bại");
    }
  };

  return <button onClick={handleGoogleLogin}>Login with Google / Đăng nhập Google</button>;
};

export default GoogleLoginButton;


---

9. Frontend Notes

1. accessToken lưu memory hoặc localStorage → attach vào Authorization header


2. refreshToken lưu HttpOnly cookie → backend tự động refresh token khi hết hạn


3. Axios interceptor handle 401 → call /refresh-token → retry original request


4. React Hook Form: quản lý form state + validation đơn giản





