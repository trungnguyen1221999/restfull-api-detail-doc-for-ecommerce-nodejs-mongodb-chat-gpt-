
---

# CHƯƠNG 13 – ADVANCED FILTER + SEARCH + PAGINATION

---

## 13.1 Filter State & Types / src/components/ProductFilter.tsx

```ts
export type ProductFilter = {
  search?: string;
  categoryId?: string;
  minPrice?: number;
  maxPrice?: number;
  rating?: number;
  topSelling?: boolean;
  sortBy?: "priceAsc" | "priceDesc" | "rating" | "newest";
  page?: number;
  limit?: number;
};
```

**Giải thích:**

* Đây là interface TypeScript cho filter API
* Có thể kết hợp tất cả: category + rating + price + search + topSelling

---

## 13.2 ProductList với Filter + Search + Pagination / src/components/ProductListAdvanced.tsx

```tsx
import React, { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { getProducts } from "../api/product";
import { ProductFilter } from "./ProductFilter";
import { getCategories } from "../api/category";

const ProductListAdvanced: React.FC = () => {
  const [filter, setFilter] = useState<ProductFilter>({ page: 1, limit: 10 });
  const { data: categories } = useQuery(["categories"], getCategories);
  
  // Fetch products với filter dynamic
  const { data: products, isLoading } = useQuery(
    ["products", filter],
    () => getProducts(filter),
    { keepPreviousData: true }
  );

  // Handle filter changes
  const handleCategoryChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setFilter(prev => ({ ...prev, categoryId: e.target.value, page: 1 }));
  };

  const handleRatingChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setFilter(prev => ({ ...prev, rating: Number(e.target.value), page: 1 }));
  };

  const handlePriceChange = (min: number, max: number) => {
    setFilter(prev => ({ ...prev, minPrice: min, maxPrice: max, page: 1 }));
  };

  const handleSearchChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFilter(prev => ({ ...prev, search: e.target.value, page: 1 }));
  };

  const handleSortChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setFilter(prev => ({ ...prev, sortBy: e.target.value as any, page: 1 }));
  };

  const handlePageChange = (newPage: number) => {
    setFilter(prev => ({ ...prev, page: newPage }));
  };

  if (isLoading) return <div>Loading...</div>;

  return (
    <div>
      {/* Filter Controls */}
      <input type="text" placeholder="Search..." onChange={handleSearchChange} />
      <select onChange={handleCategoryChange}>
        <option value="">All Categories</option>
        {categories?.map(cat => (
          <option key={cat._id} value={cat._id}>{cat.name}</option>
        ))}
      </select>
      <select onChange={handleRatingChange}>
        <option value="">All Ratings</option>
        {[5,4,3,2,1].map(r => <option key={r} value={r}>{r} stars</option>)}
      </select>
      <select onChange={handleSortChange}>
        <option value="">Sort By</option>
        <option value="priceAsc">Price Low → High</option>
        <option value="priceDesc">Price High → Low</option>
        <option value="rating">Rating</option>
        <option value="newest">Newest</option>
      </select>
      {/* Price Range */}
      <input type="number" placeholder="Min Price" onChange={e => handlePriceChange(Number(e.target.value), filter.maxPrice || 0)} />
      <input type="number" placeholder="Max Price" onChange={e => handlePriceChange(filter.minPrice || 0, Number(e.target.value))} />

      {/* Product List */}
      <div>
        {products?.items.map(p => (
          <div key={p._id}>
            <h3>{p.name}</h3>
            <p>Price: {p.price}</p>
            <p>Rating: {p.rating}</p>
          </div>
        ))}
      </div>

      {/* Pagination */}
      <div>
        {Array.from({ length: products?.totalPages || 1 }).map((_, i) => (
          <button key={i} onClick={() => handlePageChange(i + 1)}>{i + 1}</button>
        ))}
      </div>
    </div>
  );
};

export default ProductListAdvanced;
```

---

### 13.3 Giải thích chi tiết

1. **Filter kết hợp nhiều điều kiện**

* `categoryId + rating`
* `categoryId + topSelling`
* `categoryId + price min/max`
* `search + rating`
* Tất cả filter được gói trong `filter` state TypeScript

2. **React Query**

* Key: `["products", filter]` → tự động re-fetch khi filter thay đổi
* `keepPreviousData: true` → giữ data cũ khi đổi trang hoặc filter

3. **Pagination**

* Backend phải trả `{ items: Product[], totalPages: number }`
* Buttons render theo `totalPages`, bấm thay đổi `filter.page`

4. **Sort**

* `priceAsc`, `priceDesc`, `rating`, `newest`
* Kết hợp filter vẫn đúng logic

5. **Search**

* `handleSearchChange` cập nhật `filter.search` → tự động re-fetch

---

### 13.4 Kết hợp với Cart + Order

* Product list vẫn có **Add to Cart** như CHƯƠNG 12
* Sau khi chọn sản phẩm và filter, người dùng có thể add, checkout, xem order history
* Tất cả thao tác sử dụng **React Query** để fetch, mutate, invalidate

---

### ✅ Kết luận CHƯƠNG 13

* Frontend full filter nâng cao, pagination, search, sort, kết hợp nhiều điều kiện
* Dễ dàng kết hợp với **Cart + Checkout + Orders**
* Sử dụng **TypeScript**, **React Hook Form**, **TanStack Query**
* Sát thực tế **FPT Shop**, có thể copy trực tiếp vào project

---

Nếu bạn muốn, bước tiếp theo mình có thể viết **CHƯƠNG 14 – FULL PROJECT READY TO DEPLOY**, bao gồm:

* Backend + Frontend full stack
* Render (backend) + Vercel/Netlify (frontend)
* Env config + seed data + toàn bộ flow từ đăng ký → checkout → order history
* Song ngữ, chi tiết từng bước

