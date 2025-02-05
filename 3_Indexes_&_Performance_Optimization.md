# **📌 Indexes & Performance Optimization in Prisma**
Indexes are **crucial** for optimizing database performance, especially when working with large datasets. They **speed up queries** by allowing the database to find records **faster** instead of scanning the entire table.

---

## **🔹 Why Use Indexes?**
✅ **Faster Read Queries** – Indexes help in quickly retrieving data.  
✅ **Efficient Searching** – Instead of scanning all records, indexes provide direct lookup.  
✅ **Optimized Sorting & Filtering** – Queries using `ORDER BY` and `WHERE` conditions work much faster.  
✅ **Better Query Planning** – The database can use indexed fields to create efficient execution plans.  

However, **indexes also have downsides**:
❌ **More Storage Usage** – Indexes take extra space.  
❌ **Slower Inserts/Updates** – Indexes must be updated when inserting or modifying records.  

---

## **🔹 Types of Indexes in Prisma**
Prisma supports different types of indexes that help optimize query performance.

1️⃣ **Single-Column Index** – Index a single field.  
2️⃣ **Multi-Column (Compound) Index** – Index multiple fields together.  
3️⃣ **Unique Index** – Ensure values in a column are unique.  
4️⃣ **Full-Text Index (MySQL, PostgreSQL, MongoDB)** – Optimize text-based searches.  

---

# **📌 1. Single-Column Index**
### ✅ When to Use?
- When **frequently searching** by a single column.
- Example: Searching for users by email.

### **📝 Schema**
```prisma
model User {
  id    String @id @default(uuid())
  email String @unique
  name  String

  @@index([email])  // Adding an index on email
}
```

### **🛠️ Query Example**
```ts
const user = await prisma.user.findUnique({
  where: { email: "john@example.com" },
});
console.log(user);
```
🚀 **Performance Boost**: The index allows the database to **directly find** the email instead of scanning all users.

---

# **📌 2. Multi-Column (Compound) Index**
### ✅ When to Use?
- When **queries filter by multiple fields together**.
- Example: Searching orders by `customerId` **and** `status`.

### **📝 Schema**
```prisma
model Order {
  id         String @id @default(uuid())
  customerId String
  status     String

  @@index([customerId, status])  // Compound index
}
```

### **🛠️ Query Example**
```ts
const orders = await prisma.order.findMany({
  where: {
    customerId: "123",
    status: "shipped",
  },
});
console.log(orders);
```
🚀 **Performance Boost**: The compound index helps quickly **filter orders for a specific customer** and status.

---

# **📌 3. Unique Index**
### ✅ When to Use?
- When a column must **always have unique values**.
- Example: Ensuring usernames are **unique**.

### **📝 Schema**
```prisma
model User {
  id       String @id @default(uuid())
  username String @unique
}
```

### **🛠️ Query Example**
```ts
const user = await prisma.user.create({
  data: {
    username: "john_doe",
  },
});
console.log(user);
```
🚀 **Prevention**: Prisma **automatically enforces uniqueness**, preventing duplicate usernames.

---

# **📌 4. Full-Text Search Index (MySQL, PostgreSQL, MongoDB)**
### ✅ When to Use?
- When searching **large text fields**.
- Example: Searching blog posts by title **and** content.

### **📝 Schema**
```prisma
model Post {
  id      String @id @default(uuid())
  title   String
  content String

  @@fulltext([title, content])  // Full-text search index
}
```

### **🛠️ Query Example**
```ts
const posts = await prisma.post.findMany({
  where: {
    OR: [
      { title: { contains: "prisma" } },
      { content: { contains: "prisma" } },
    ],
  },
});
console.log(posts);
```
🚀 **Faster Text Search**: This enables **efficient keyword searching** across **large text fields**.

---

# **📌 5. Indexing Foreign Keys**
### ✅ When to Use?
- When filtering by **foreign key relationships**.
- Example: Finding all **posts by a user**.

### **📝 Schema**
```prisma
model User {
  id    String @id @default(uuid())
  name  String
  posts Post[]
}

model Post {
  id      String @id @default(uuid())
  title   String
  userId  String

  @@index([userId])  // Index on foreign key
}
```

### **🛠️ Query Example**
```ts
const userPosts = await prisma.post.findMany({
  where: { userId: "123" },
});
console.log(userPosts);
```
🚀 **Performance Boost**: This allows **faster lookups** of posts by a specific user.

---

# **📌 6. Optimizing Sorting Queries with Indexes**
### ✅ When to Use?
- When **sorting large datasets**.

### **📝 Schema**
```prisma
model Product {
  id        String  @id @default(uuid())
  name      String
  price     Float
  createdAt DateTime @default(now())

  @@index([createdAt])  // Index for sorting
}
```

### **🛠️ Query Example**
```ts
const products = await prisma.product.findMany({
  orderBy: { createdAt: "desc" }, // Sort by newest products
});
console.log(products);
```
🚀 **Faster Sorting**: Sorting by `createdAt` is much **quicker** with an index.

---

# **📌 Best Practices for Indexing**
✅ **Index frequently queried fields** (`WHERE`, `ORDER BY`, `JOIN` conditions).  
✅ **Use compound indexes** when queries filter by **multiple fields together**.  
✅ **Always index foreign keys** to improve **JOIN performance**.  
✅ **Avoid over-indexing** to **reduce storage costs** and improve **insert/update speeds**.  
✅ **Use full-text indexes** for **searching text-heavy fields** like blogs.  
✅ **Test performance using EXPLAIN ANALYZE** (SQL) or **profiling queries** (MongoDB).  

---

# **📌 Example: Real-World Use Case**
## 🚀 **E-Commerce Application Optimization**
Let's say we are building an **e-commerce platform** with **users, orders, and products**.  
A common query is: **Find all shipped orders for a user, sorted by date.**

### **📝 Optimized Schema**
```prisma
model User {
  id    String @id @default(uuid())
  email String @unique
  orders Order[]
}

model Order {
  id         String  @id @default(uuid())
  userId     String
  status     String
  createdAt  DateTime @default(now())

  @@index([userId, status])   // Compound index on userId and status
  @@index([createdAt])        // Index for sorting
}
```

### **🛠️ Query Example**
```ts
const orders = await prisma.order.findMany({
  where: {
    userId: "123",
    status: "shipped",
  },
  orderBy: { createdAt: "desc" }, // Sort by newest
});
console.log(orders);
```
### **🚀 Performance Benefits**
✅ **Faster lookup** because the **userId + status** index optimizes filtering.  
✅ **Faster sorting** because the **createdAt** index optimizes ordering.  
✅ **Better user experience** with **instant results**.  

---

# **📌 Final Thoughts**
- **Indexes are essential** for optimizing Prisma applications.  
- **Use them wisely** to avoid overloading the database.  
- **Test with real data** to see performance improvements.  

---
