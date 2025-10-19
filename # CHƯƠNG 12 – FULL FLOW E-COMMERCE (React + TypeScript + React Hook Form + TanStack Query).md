
---

# CHƯƠNG 12 – FULL FLOW E-COMMERCE (React + TypeScript + React Hook Form + TanStack Query)

---

## 12.1 Cài đặt thư viện

```bash
npm install react-hook-form @tanstack/react-query axios react-router-dom
```

* `react-hook-form` → quản lý form, validation
* `@tanstack/react-query` → fetch, cache, sync data với backend
* `axios` → gọi API
* `react-router-dom` → routing

---

## 12.2 Thiết lập React Query Provider / src/main.tsx

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import App from "./App";

const queryClient = new QueryClient();

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </React.StrictMode>
);
```

**Giải thích:**

* `QueryClientProvider` bọc toàn app để sử dụng React Query
* `queryClient` quản lý cache, fetch

---

## 12.3 Auth Flow – Register & Login

### 12.3.1 Form Register / src/components/Register.tsx

```tsx
import React from "react";
import { useForm } from "react-hook-form";
import { useMutation } from "@tanstack/react-query";
import { register as registerApi } from "../api/auth";

type RegisterForm = {
  username: string;
  email: string;
  password: string;
  phone?: string;
  dob?: string;
};

const Register: React.FC = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<RegisterForm>();

  const mutation = useMutation(registerApi, {
    onSuccess: (data) => {
      alert("Register successful! Verify email: " + data.token);
    },
    onError: () => alert("Register failed")
  });

  const onSubmit = (data: RegisterForm) => {
    mutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("username", { required: true })} placeholder="Username" />
      {errors.username && <span>Username required</span>}
      
      <input {...register("email", { required: true })} placeholder="Email" />
      {errors.email && <span>Email required</span>}
      
      <input type="password" {...register("password", { required: true })} placeholder="Password" />
      {errors.password && <span>Password required</span>}
      
      <input {...register("phone")} placeholder="Phone" />
      <input type="date" {...register("dob")} placeholder="Date of Birth" />

      <button type="submit">Register</button>
    </form>
  );
};

export default Register;
```

**Giải thích:**

* `useForm` quản lý form + validation
* `useMutation` của React Query gửi dữ liệu lên backend
* `onSuccess` hiển thị token verify email

---

### 12.3.2 Form Login / src/components/Login.tsx

```tsx
import React from "react";
import { useForm } from "react-hook-form";
import { useMutation } from "@tanstack/react-query";
import { login as loginApi } from "../api/auth";

type LoginForm = { email: string; password: string };

const Login: React.FC = () => {
  const { register, handleSubmit } = useForm<LoginForm>();

  const mutation = useMutation(loginApi, {
    onSuccess: (data) => {
      localStorage.setItem("token", data.token);
      alert("Login successful!");
    },
    onError: () => alert("Login failed")
  });

  const onSubmit = (data: LoginForm) => mutation.mutate(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} placeholder="Email" />
      <input type="password" {...register("password")} placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
};

export default Login;
```

---

## 12.4 Product List + Add to Cart

### 12.4.1 Fetch Products / src/components/ProductList.tsx

```tsx
import React from "react";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getProducts } from "../api/product";
import { addToCart } from "../api/cart";

const ProductList: React.FC = () => {
  const queryClient = useQueryClient();

  const { data: products, isLoading } = useQuery(["products"], getProducts);

  const addMutation = useMutation(addToCart, {
    onSuccess: () => queryClient.invalidateQueries(["cart"])
  });

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {products?.map(p => (
        <div key={p._id}>
          <h3>{p.name}</h3>
          <p>Price: {p.price}</p>
          <button onClick={() => addMutation.mutate(p._id, 1)}>Add to Cart</button>
        </div>
      ))}
    </div>
  );
};

export default ProductList;
```

**Giải thích:**

* `useQuery` fetch data từ backend, cache tự động
* `useMutation` add product vào cart
* `invalidateQueries` refresh cart data sau khi mutate

---

## 12.5 Cart + Checkout

### 12.5.1 Cart Component / src/components/Cart.tsx

```tsx
import React from "react";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { getCart, removeFromCart, updateCartItem } from "../api/cart";
import { createOrder } from "../api/order";

const Cart: React.FC = () => {
  const queryClient = useQueryClient();
  const { data: cart, isLoading } = useQuery(["cart"], getCart);

  const removeMutation = useMutation(removeFromCart, {
    onSuccess: () => queryClient.invalidateQueries(["cart"])
  });

  const updateMutation = useMutation(updateCartItem, {
    onSuccess: () => queryClient.invalidateQueries(["cart"])
  });

  const checkoutMutation = useMutation(() => createOrder("credit_card"), {
    onSuccess: () => {
      alert("Order created!");
      queryClient.invalidateQueries(["cart"]);
      queryClient.invalidateQueries(["orders"]);
    }
  });

  if (isLoading) return <div>Loading cart...</div>;

  return (
    <div>
      {cart?.items.map(item => (
        <div key={item.product._id}>
          <span>{item.product.name}</span>
          <input
            type="number"
            value={item.quantity}
            onChange={e => updateMutation.mutate(item.product._id, Number(e.target.value))}
          />
          <button onClick={() => removeMutation.mutate(item.product._id)}>Remove</button>
        </div>
      ))}
      <p>Total: {cart?.totalPrice}</p>
      <button onClick={() => checkoutMutation.mutate()}>Checkout</button>
    </div>
  );
};

export default Cart;
```

---

## 12.6 Order History

```tsx
import React from "react";
import { useQuery } from "@tanstack/react-query";
import { getUserOrders } from "../api/order";

const Orders: React.FC = () => {
  const { data: orders, isLoading } = useQuery(["orders"], getUserOrders);

  if (isLoading) return <div>Loading orders...</div>;

  return (
    <div>
      {orders?.map(order => (
        <div key={order._id}>
          <h4>Order #{order._id} - Status: {order.status}</h4>
          {order.items.map(item => (
            <p key={item.product._id}>{item.product.name} x {item.quantity}</p>
          ))}
          <p>Total: {order.totalPrice}</p>
        </div>
      ))}
    </div>
  );
};

export default Orders;
```

---

## 12.7 Router / src/App.tsx

```tsx
import React from "react";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import Register from "./components/Register";
import Login from "./components/Login";
import ProductList from "./components/ProductList";
import Cart from "./components/Cart";
import Orders from "./components/Orders";

const App: React.FC = () => (
  <Router>
    <Routes>
      <Route path="/register" element={<Register />} />
      <Route path="/login" element={<Login />} />
      <Route path="/" element={<ProductList />} />
      <Route path="/cart" element={<Cart />} />
      <Route path="/orders" element={<Orders />} />
    </Routes>
  </Router>
);

export default App;
```

---

### ✅ Kết luận CHƯƠNG 12

* Full flow e-commerce:

  1. **Register → Verify Email → Login**
  2. **View Products → Add to Cart**
  3. **Checkout → Create Order → View Order History**
* Sử dụng **React Hook Form** cho tất cả form
* **TanStack Query** quản lý fetch + cache + mutations
* Sát thực tế **FPT Shop**
* Tất cả bằng **TypeScript**

---

