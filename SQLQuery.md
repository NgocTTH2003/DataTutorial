# SQL TUTORIAL

## Mục lục

1. [Các câu lệnh SQL quan trọng](#1-các-câu-lệnh-sql-quan-trọng)
2. [SELECT](#2-select)
3. [WHERE — Lọc bản ghi](#3-where--lọc-bản-ghi)
4. [ORDER BY — Sắp xếp](#4-order-by--sắp-xếp)
5. [AND, OR, NOT — Toán tử logic](#5-and-or-not--toán-tử-logic)
6. [INSERT INTO — Chèn dữ liệu](#6-insert-into--chèn-dữ-liệu)
7. [OUTPUT — "Biên nhận" sau khi thao tác dữ liệu](#7-output--biên-nhận-sau-khi-thao-tác-dữ-liệu)
8. [INSERT INTO SELECT — Chèn từ bảng khác](#8-insert-into-select--chèn-từ-bảng-khác)
9. [UPDATE — Cập nhật dữ liệu (DML)](#9-update--cập-nhật-dữ-liệu-dml)
10. [Xóa dữ liệu — DELETE, TRUNCATE, DROP](#10-xóa-dữ-liệu--delete-truncate-drop)
11. [DDL — Định nghĩa cấu trúc (CREATE, ALTER, DROP, TRUNCATE)](#11-ddl--định-nghĩa-cấu-trúc-create-alter-drop-truncate)
12. [DML — Thao tác dữ liệu (SELECT, INSERT, UPDATE, DELETE)](#12-dml--thao-tác-dữ-liệu-select-insert-update-delete)
13. [Constraints — Ràng buộc](#13-constraints--ràng-buộc)

---

## 1. Các câu lệnh SQL quan trọng

SQL được chia thành **5 nhóm** chính:

| Nhóm | Tên đầy đủ | Thao tác với | Các lệnh |
|---|---|---|---|
| **DDL** | Data Definition Language | Cấu trúc (bảng, cột, index) | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data Manipulation Language | Dữ liệu (hàng) | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language | Quyền truy cập | `GRANT`, `REVOKE` |
| **TCL** | Transaction Control Language | Giao dịch | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |
| **DQL** | Data Query Language | Truy vấn | `SELECT` (một số tài liệu tách riêng khỏi DML) |

> **Dễ nhớ:** DDL thay đổi **cái hộp** (cấu trúc), DML thay đổi **đồ bên trong hộp** (dữ liệu).

### Tổng hợp các câu lệnh thường dùng

| Câu lệnh | Nhóm | Mục đích |
|---|---|---|
| `SELECT` | DML | Trích xuất dữ liệu từ cơ sở dữ liệu |
| `INSERT INTO` | DML | Chèn dữ liệu mới vào cơ sở dữ liệu |
| `UPDATE` | DML | Cập nhật dữ liệu trong cơ sở dữ liệu |
| `DELETE` | DML | Xóa dữ liệu khỏi cơ sở dữ liệu |
| `CREATE DATABASE / TABLE` | DDL | Tạo cơ sở dữ liệu / bảng mới |
| `ALTER DATABASE / TABLE` | DDL | Chỉnh sửa cơ sở dữ liệu / bảng |
| `DROP TABLE` | DDL | Xóa một bảng (cả cấu trúc + dữ liệu) |
| `TRUNCATE TABLE` | DDL | Xóa toàn bộ dữ liệu, giữ cấu trúc |
| `CREATE INDEX` | DDL | Tạo chỉ mục (khóa tìm kiếm) |
| `DROP INDEX` | DDL | Xóa một chỉ mục |
| `GRANT` | DCL | Cấp quyền cho user |
| `REVOKE` | DCL | Thu hồi quyền |
| `COMMIT` | TCL | Xác nhận giao dịch |
| `ROLLBACK` | TCL | Hủy giao dịch |

---

## 2. SELECT

```sql
-- Lấy tất cả cột
SELECT * FROM table_name;

-- Lấy cột cụ thể
SELECT col1, col2 FROM table_name;

-- Lấy giá trị không trùng lặp
SELECT DISTINCT col FROM table_name;

-- Đếm số giá trị không trùng lặp
SELECT COUNT(DISTINCT Country) FROM Customers;
```

---

## 3. WHERE — Lọc bản ghi

> **Lưu ý:** Giá trị văn bản phải đặt trong dấu nháy đơn `' '`

```sql
SELECT col1, col2 FROM table_name
WHERE condition;
```

**Các toán tử:** `=` `>` `<` `>=` `<=` `<>` `BETWEEN` `LIKE` `IN`

```sql
-- BETWEEN: lọc trong khoảng
SELECT * FROM Products
WHERE Price BETWEEN 50 AND 60;

-- LIKE: tìm theo mẫu
SELECT * FROM Customers
WHERE City LIKE 's%';

-- IN: lọc theo danh sách
SELECT * FROM Customers
WHERE Country IN ('Germany', 'France');
```

---

## 4. ORDER BY — Sắp xếp

- `ASC` = tăng dần (mặc định)
- `DESC` = giảm dần

```sql
SELECT * FROM Customers
ORDER BY Country ASC, CustomerName DESC;
```

---

## 5. AND, OR, NOT — Toán tử logic

```sql
-- NOT: phủ định điều kiện
SELECT * FROM Customers
WHERE NOT Country = 'Spain';
```

Các dạng mở rộng: `NOT LIKE`, `NOT BETWEEN`, `NOT IN`, `IS NOT NULL`, `NOT EXISTS`

---

## 6. INSERT INTO — Chèn dữ liệu

**Cú pháp 1:** Chỉ định cột

```sql
INSERT INTO table_name (col1, col2, col3)
VALUES (val1, val2, val3);
```

**Cú pháp 2:** Chèn tất cả cột (theo đúng thứ tự)

```sql
INSERT INTO table_name
VALUES (val1, val2, val3);
```

### 6.1. Chèn nhiều hàng

```sql
INSERT INTO sales.promotions (promotion_name, discount, start_date, expired_date)
VALUES
    ('2020 Summer Promotion', 0.25, '20200601', '20200901'),
    ('2020 Fall Promotion', 0.10, '20201001', '20201101'),
    ('2020 Winter Promotion', 0.25, '20201201', '20210101');
```

### 6.2. Chèn vào cột IDENTITY

Cột IDENTITY tự tăng, không cho phép chèn thủ công. Muốn chèn thì phải bật/tắt:

```sql
SET IDENTITY_INSERT sales.promotions ON;

INSERT INTO sales.promotions (promotion_id, promotion_name, discount, start_date, expired_date)
VALUES (4, '2019 Spring Promotion', 0.25, '20190201', '20190301');

SET IDENTITY_INSERT sales.promotions OFF;
```

---

## 7. OUTPUT — "Biên nhận" sau khi thao tác dữ liệu

> **OUTPUT** giống như hóa đơn: câu lệnh vẫn chạy bình thường nếu không có, nhưng có thì SQL Server tự trả kết quả luôn, không cần `SELECT` lại.

```sql
-- INSERT: xem dòng vừa thêm
INSERT INTO sales.promotions (promotion_name, discount, start_date, expired_date)
OUTPUT inserted.promotion_id, inserted.promotion_name
VALUES ('2020 Summer Promotion', 0.25, '20200601', '20200901');

-- UPDATE: xem giá trị trước và sau
UPDATE sales.promotions SET discount = 0.30
OUTPUT deleted.discount AS gia_cu, inserted.discount AS gia_moi
WHERE promotion_id = 1;

-- DELETE: xem dòng vừa xóa
DELETE FROM sales.promotions
OUTPUT deleted.promotion_name, deleted.discount
WHERE expired_date < '20210101';
```

---

## 8. INSERT INTO SELECT — Chèn từ bảng khác

> **Yêu cầu:** Bảng đích phải tồn tại trước.

```sql
-- Chèn tất cả dữ liệu phù hợp
INSERT INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers
ORDER BY first_name, last_name;

-- Chèn top N dòng
INSERT TOP (10)
INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers
ORDER BY first_name, last_name;

-- Chèn top N phần trăm
INSERT TOP (10) PERCENT
INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers
ORDER BY first_name, last_name;
```

---

## 9. UPDATE — Cập nhật dữ liệu (DML)

### 9.1. UPDATE cơ bản

```sql
UPDATE table_name
SET col1 = val1, col2 = val2
WHERE condition;
```

> **Lưu ý:** Nếu không có `WHERE`, toàn bộ bảng sẽ bị cập nhật!

### 9.2. UPDATE với JOIN

```sql
-- Cập nhật bảng A dựa trên dữ liệu từ bảng B
UPDATE a
SET a.price = b.new_price
FROM products a
INNER JOIN price_updates b ON a.id = b.product_id;
```

### 9.3. UPDATE với Subquery

```sql
-- Cập nhật giá bằng giá trung bình của category A
UPDATE products
SET price = (SELECT AVG(price) FROM products WHERE category = 'A')
WHERE category = 'B';
```

### 9.4. UPDATE với CTE

```sql
-- Giảm 10% sản phẩm đắt nhất mỗi loại
WITH cte AS (
    SELECT id, price,
           ROW_NUMBER() OVER(PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
)
UPDATE cte
SET price = price * 0.9
WHERE rn = 1;
```

### 9.5. UPDATE TOP

```sql
-- Cập nhật N dòng đầu tiên
UPDATE TOP (10) products
SET price = price * 1.1
WHERE category = 'A';
```

### 9.6. UPDATE với OUTPUT

```sql
-- Xem giá trị trước và sau khi cập nhật
UPDATE products
SET price = price * 1.1
OUTPUT deleted.price AS gia_cu, inserted.price AS gia_moi
WHERE category = 'A';
```

---

## 10. Xóa dữ liệu — DELETE, TRUNCATE, DROP

### 10.1. DELETE (DML)

Xóa dữ liệu các hàng trong bảng nhưng bảng và cấu trúc của nó vẫn được giữ nguyên.

**Đặc điểm:**
- Xóa từng hàng một → chậm hơn TRUNCATE
- Kích hoạt Trigger
- Có thể dùng `WHERE` để xóa có điều kiện
- Có thể xóa dữ liệu trong bảng được tham chiếu bởi khóa ngoại nếu sử dụng cấu hình `ON DELETE CASCADE` hoặc xóa các bảng con trước

```sql
-- Xóa có điều kiện
DELETE FROM table_name
WHERE condition;

-- Xóa toàn bộ dữ liệu (giữ cấu trúc bảng)
DELETE FROM table_name;
```

> **Lưu ý:** Nếu không có `WHERE`, toàn bộ dữ liệu trong bảng sẽ bị xóa!

**DELETE TOP:**

```sql
-- Xóa N dòng đầu tiên
DELETE TOP (5) FROM orders
WHERE status = 'Cancelled';
```

**DELETE với JOIN:**

```sql
-- Xóa dòng trong bảng A dựa trên điều kiện từ bảng B
DELETE a
FROM orders a
INNER JOIN customers b ON a.customer_id = b.id
WHERE b.status = 'Inactive';
```

**DELETE với OUTPUT:**

```sql
-- Xem dòng vừa xóa
DELETE FROM products
OUTPUT deleted.id, deleted.name, deleted.price
WHERE price < 10;
```

### 10.2. TRUNCATE (DDL)

Xóa toàn bộ dữ liệu trong bảng nhanh chóng nhưng vẫn giữ nguyên cấu trúc bảng (schema), chỉ mục và ràng buộc.

**Đặc điểm:**
- Nhanh hơn DELETE (xóa toàn bộ trang dữ liệu thay vì từng hàng)
- Không sử dụng `WHERE`
- Không kích hoạt Trigger
- Không thể thực hiện nếu bảng được tham chiếu bởi khóa ngoại
- Reset IDENTITY về giá trị seed ban đầu

```sql
TRUNCATE TABLE table_name;
```

### 10.3. DROP (DDL)

Xóa toàn bộ bảng khỏi CSDL bao gồm cả cấu trúc, chỉ mục và ràng buộc.

```sql
-- Xóa bảng
DROP TABLE table_name;

-- Xóa bảng nếu tồn tại (tránh lỗi)
DROP TABLE IF EXISTS table_name;
```

### 10.4. So sánh DELETE vs TRUNCATE vs DROP

| Tiêu chí | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Nhóm | **DML** | **DDL** | **DDL** |
| Xóa gì | Dữ liệu (có WHERE) | Toàn bộ dữ liệu | Cả bảng + cấu trúc |
| Giữ cấu trúc bảng | ✅ Có | ✅ Có | ❌ Không |
| Giữ chỉ mục, ràng buộc | ✅ Có | ✅ Có | ❌ Không |
| Dùng WHERE | ✅ Có | ❌ Không | ❌ Không |
| Kích hoạt Trigger | ✅ Có | ❌ Không | ❌ Không |
| Tốc độ | 🐢 Chậm (xóa từng hàng) | 🐇 Nhanh | 🐇 Nhanh |
| Reset IDENTITY | ❌ Không | ✅ Có (reset về seed) | — |
| Khóa ngoại (FK) | ✅ Được (nếu có CASCADE) | ❌ Không được nếu có FK | ❌ Không được nếu có FK |
| Rollback | ✅ Có | ✅ Có (trong transaction) | ✅ Có (trong transaction) |

---

## 11. DDL — Định nghĩa cấu trúc (CREATE, ALTER, DROP, TRUNCATE)

> **Nhắc lại:** DDL (Data Definition Language) thao tác với **cấu trúc** database, không phải dữ liệu bên trong.

### 11.1. CREATE — Tạo đối tượng

```sql
-- Tạo database
CREATE DATABASE my_database;

-- Tạo bảng
CREATE TABLE employees (
    id INT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    salary DECIMAL(18,2) DEFAULT 0,
    department_id INT,
    created_at DATETIME2 DEFAULT GETDATE()
);

-- Tạo bảng từ bảng khác (copy cấu trúc + data)
SELECT * INTO new_table FROM old_table;

-- Tạo bảng chỉ copy cấu trúc (không có data)
SELECT * INTO new_table FROM old_table WHERE 1 = 0;

-- Tạo schema
CREATE SCHEMA sales;

-- Tạo index
CREATE INDEX idx_name ON employees(name);
```

### 11.2. ALTER — Chỉnh sửa đối tượng

**Thêm cột:**

```sql
ALTER TABLE employees ADD phone VARCHAR(20);

-- Thêm nhiều cột cùng lúc
ALTER TABLE employees
ADD address NVARCHAR(255),
    date_of_birth DATE;
```

**Xóa cột:**

```sql
ALTER TABLE employees DROP COLUMN phone;
```

**Thay đổi kiểu dữ liệu:**

```sql
ALTER TABLE employees ALTER COLUMN name NVARCHAR(200);
```

**Thêm / xóa ràng buộc:**

```sql
-- Thêm ràng buộc
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary >= 0);

ALTER TABLE employees
ADD CONSTRAINT df_status DEFAULT 'Active' FOR status;

-- Xóa ràng buộc
ALTER TABLE employees
DROP CONSTRAINT chk_salary;
```

**Đổi tên (dùng sp_rename):**

```sql
-- Đổi tên bảng
EXEC sp_rename 'old_table', 'new_table';

-- Đổi tên cột
EXEC sp_rename 'employees.phone', 'phone_number', 'COLUMN';

-- Đổi tên index
EXEC sp_rename 'employees.idx_old', 'idx_new', 'INDEX';
```

> **Lưu ý:** SQL Server không có lệnh `RENAME` riêng, phải dùng stored procedure `sp_rename`.

### 11.3. DROP — Xóa đối tượng

```sql
-- Xóa bảng
DROP TABLE table_name;

-- Xóa bảng nếu tồn tại (tránh lỗi)
DROP TABLE IF EXISTS table_name;

-- Xóa database
DROP DATABASE my_database;

-- Xóa index
DROP INDEX idx_name ON table_name;

-- Xóa view
DROP VIEW vw_name;

-- Xóa stored procedure
DROP PROCEDURE sp_name;
```

> **Lưu ý:** `DROP` xóa hoàn toàn đối tượng khỏi database (cả cấu trúc + dữ liệu). Không thể phục hồi nếu không có backup!

### 11.4. TRUNCATE — Xóa toàn bộ dữ liệu

```sql
TRUNCATE TABLE table_name;
```

> **Lưu ý:** Xem so sánh DELETE vs TRUNCATE vs DROP chi tiết ở mục 10.5.

### Tổng hợp DDL

| Lệnh | Mục đích | Xóa cấu trúc? | Xóa dữ liệu? |
|---|---|---|---|
| `CREATE` | Tạo mới database, table, index, view... | — | — |
| `ALTER` | Thêm/xóa/sửa cột, ràng buộc, đổi tên | ❌ | ❌ |
| `DROP` | Xóa hoàn toàn đối tượng | ✅ | ✅ |
| `TRUNCATE` | Xóa toàn bộ dữ liệu, giữ cấu trúc | ❌ | ✅ |
---

## 12. DML — Thao tác dữ liệu (SELECT, INSERT, UPDATE, DELETE)

> **Nhắc lại:** DML (Data Manipulation Language) thao tác với **dữ liệu bên trong bảng**, không thay đổi cấu trúc.

### 12.1. Tổng hợp 4 lệnh DML

| Lệnh | Mục đích | Cần WHERE? | Kích hoạt Trigger? |
|---|---|---|---|
| `SELECT` | Đọc dữ liệu | Tùy chọn | ❌ Không |
| `INSERT` | Thêm dữ liệu mới | ❌ Không | ✅ Có |
| `UPDATE` | Sửa dữ liệu hiện tại | ⚠️ Nên có (không có = sửa hết) | ✅ Có |
| `DELETE` | Xóa dữ liệu | ⚠️ Nên có (không có = xóa hết) | ✅ Có |

### 12.2. Thứ tự thực thi câu lệnh SELECT

Thứ tự **viết** khác với thứ tự SQL Server **thực thi**. Bài test hay hỏi cái này:

```
Thứ tự viết:          Thứ tự thực thi:
SELECT        (5)     FROM          (1)
FROM          (1)     WHERE         (2)
WHERE         (2)     GROUP BY      (3)
GROUP BY      (3)     HAVING        (4)
HAVING        (4)     SELECT        (5)
ORDER BY      (6)     ORDER BY      (6)
```

> **Dễ nhớ:** SQL đọc **FROM** trước (biết bảng nào) → **WHERE** (lọc hàng) → **GROUP BY** (nhóm) → **HAVING** (lọc nhóm) → **SELECT** (chọn cột) → **ORDER BY** (sắp xếp cuối cùng).

### 12.3. Tổng hợp các cách dùng từng lệnh

**SELECT** — Chi tiết ở mục 2, 3, 4, 5

```sql
SELECT DISTINCT col1, col2      -- Chọn cột, loại trùng
FROM table_name                  -- Từ bảng nào
WHERE condition                  -- Lọc hàng
GROUP BY col1                    -- Nhóm
HAVING COUNT(*) > 5              -- Lọc sau nhóm
ORDER BY col1 DESC;              -- Sắp xếp
```

**INSERT** — Chi tiết ở mục 6, 8

```sql
-- Chèn 1 dòng
INSERT INTO table_name (col1, col2) VALUES (val1, val2);

-- Chèn nhiều dòng
INSERT INTO table_name (col1, col2)
VALUES (val1, val2), (val3, val4), (val5, val6);

-- Chèn từ bảng khác
INSERT INTO table_a (col1, col2)
SELECT col1, col2 FROM table_b WHERE condition;
```

**UPDATE** — Chi tiết ở mục 9

```sql
-- Cơ bản
UPDATE table_name SET col1 = val1 WHERE condition;

-- Với JOIN
UPDATE a SET a.col1 = b.col1
FROM table_a a INNER JOIN table_b b ON a.id = b.id;

-- Với Subquery
UPDATE table_name SET col1 = (SELECT AVG(col1) FROM table_name) WHERE condition;

-- Với CTE
WITH cte AS (SELECT id, ROW_NUMBER() OVER(ORDER BY id) AS rn FROM table_name)
UPDATE cte SET col1 = val1 WHERE rn = 1;

-- TOP
UPDATE TOP (10) table_name SET col1 = val1 WHERE condition;
```

**DELETE** — Chi tiết ở mục 10

```sql
-- Cơ bản
DELETE FROM table_name WHERE condition;

-- TOP
DELETE TOP (5) FROM table_name WHERE condition;

-- Với JOIN
DELETE a FROM table_a a INNER JOIN table_b b ON a.id = b.id WHERE b.status = 'Inactive';
```

### 12.4. DML vs DDL — So sánh nhanh

| Tiêu chí | DML | DDL |
|---|---|---|
| Thao tác với | Dữ liệu (hàng) | Cấu trúc (bảng, cột, index) |
| Các lệnh | SELECT, INSERT, UPDATE, DELETE | CREATE, ALTER, DROP, TRUNCATE |
| Rollback | ✅ Có (trong transaction) | ✅ Có (trong transaction) |
| Kích hoạt Trigger | ✅ Có (DML Trigger) | ✅ Có (DDL Trigger) |
| Ví dụ | Thêm/sửa/xóa 1 dòng khách hàng | Thêm cột mới vào bảng khách hàng |

> **Lưu ý bài test:**
> - `TRUNCATE` thuộc **DDL** (không phải DML) — vì nó thao tác ở cấp bảng, không xóa từng hàng
> - `SELECT INTO` vừa có tính DML (copy data) vừa có tính DDL (tạo bảng mới)
> - `OUTPUT` có thể dùng với cả INSERT, UPDATE, DELETE — xem mục 7
---

## 13. Constraints — Ràng buộc

Constraints là các quy tắc áp dụng lên cột hoặc bảng để đảm bảo tính chính xác và toàn vẹn của dữ liệu.

### 13.1. Tổng hợp 6 loại Constraints

| Constraint | Mục đích | Cho phép NULL? | Số lượng / bảng |
|---|---|---|---|
| `PRIMARY KEY` | Duy nhất + không NULL | ❌ Không | Chỉ 1 |
| `UNIQUE` | Duy nhất | ✅ Có (tối đa 1 NULL) | Nhiều |
| `NOT NULL` | Không cho phép NULL | — | Nhiều |
| `CHECK` | Kiểm tra điều kiện | — | Nhiều |
| `DEFAULT` | Giá trị mặc định khi không chỉ định | — | Nhiều |
| `FOREIGN KEY` | Tham chiếu bảng khác | ✅ Có | Nhiều |

### 13.2. PRIMARY KEY — Khóa chính

Mỗi bảng chỉ có **1** PRIMARY KEY. Giá trị phải **duy nhất** và **không được NULL**.

```sql
-- Cách 1: Khai báo inline (1 cột)
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name NVARCHAR(100)
);

-- Cách 2: Khai báo cuối bảng (1 hoặc nhiều cột — composite key)
CREATE TABLE order_details (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- Cách 3: Thêm sau bằng ALTER
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (id);
```

> **Lưu ý:** PRIMARY KEY tự động tạo **Clustered Index** (mặc định).

### 13.3. UNIQUE — Duy nhất

Đảm bảo giá trị trong cột là duy nhất. Khác PRIMARY KEY ở chỗ **cho phép NULL** (tối đa 1 hàng NULL).

```sql
-- Khai báo khi tạo bảng
CREATE TABLE employees (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);

-- Thêm sau bằng ALTER
ALTER TABLE employees
ADD CONSTRAINT uq_email UNIQUE (email);
```

### 13.4. NOT NULL — Không cho phép NULL

Bắt buộc cột phải có giá trị, không được bỏ trống.

```sql
-- Khai báo khi tạo bảng
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name NVARCHAR(100) NOT NULL,
    email VARCHAR(100)              -- cho phép NULL (mặc định)
);

-- Thêm sau bằng ALTER
ALTER TABLE employees
ALTER COLUMN name NVARCHAR(100) NOT NULL;
```

### 13.5. CHECK — Kiểm tra điều kiện

Chỉ cho phép giá trị thỏa mãn biểu thức Boolean.

```sql
-- Khai báo khi tạo bảng
CREATE TABLE employees (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 18),
    salary DECIMAL(18,2) CHECK (salary > 0)
);

-- Thêm sau bằng ALTER
ALTER TABLE employees
ADD CONSTRAINT chk_age CHECK (age >= 18);

-- CHECK với nhiều điều kiện
ALTER TABLE employees
ADD CONSTRAINT chk_salary_range CHECK (salary >= 0 AND salary <= 1000000000);
```

### 13.6. DEFAULT — Giá trị mặc định

Tự động gán giá trị khi INSERT mà không chỉ định cột đó.

```sql
-- Khai báo khi tạo bảng
CREATE TABLE employees (
    id INT PRIMARY KEY,
    status NVARCHAR(20) DEFAULT 'Active',
    created_at DATETIME2 DEFAULT GETDATE()
);

-- Thêm sau bằng ALTER
ALTER TABLE employees
ADD CONSTRAINT df_status DEFAULT 'Active' FOR status;
```

> **Lưu ý:** Nếu INSERT có chỉ định giá trị cho cột thì DEFAULT bị bỏ qua. DEFAULT chỉ áp dụng khi cột không được nhắc đến trong INSERT.

### 13.7. FOREIGN KEY — Khóa ngoại

Tạo mối quan hệ giữa 2 bảng. Giá trị ở cột FK phải tồn tại trong cột PK/UNIQUE của bảng được tham chiếu.

```sql
-- Khai báo khi tạo bảng
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name NVARCHAR(100)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name NVARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Thêm sau bằng ALTER
ALTER TABLE employees
ADD CONSTRAINT fk_dept FOREIGN KEY (department_id) REFERENCES departments(id);
```

**Các hành vi khi xóa/cập nhật bản ghi cha (ON DELETE / ON UPDATE):**

| Tùy chọn | Khi xóa/sửa bản ghi cha |
|---|---|
| `NO ACTION` | Lỗi — không cho xóa/sửa nếu có bản ghi con (mặc định) |
| `CASCADE` | Tự động xóa/sửa bản ghi con theo |
| `SET NULL` | Gán NULL cho cột FK ở bản ghi con |
| `SET DEFAULT` | Gán giá trị DEFAULT cho cột FK ở bản ghi con |

```sql
-- Ví dụ: xóa department → tự động xóa employees trong department đó
ALTER TABLE employees
ADD CONSTRAINT fk_dept FOREIGN KEY (department_id)
REFERENCES departments(id)
ON DELETE CASCADE
ON UPDATE CASCADE;

-- Ví dụ: xóa department → gán NULL cho department_id của employees
ALTER TABLE employees
ADD CONSTRAINT fk_dept FOREIGN KEY (department_id)
REFERENCES departments(id)
ON DELETE SET NULL;
```

### 13.8. Xóa Constraints

```sql
-- Xóa bất kỳ constraint nào (PK, FK, UNIQUE, CHECK, DEFAULT)
ALTER TABLE employees
DROP CONSTRAINT constraint_name;

-- Xem danh sách constraints của bảng
SELECT * FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'employees';
```

### 13.9. PRIMARY KEY vs UNIQUE — So sánh

| Tiêu chí | PRIMARY KEY | UNIQUE |
|---|---|---|
| Số lượng / bảng | Chỉ 1 | Nhiều |
| Cho phép NULL | ❌ Không | ✅ Có (tối đa 1 NULL) |
| Tạo Index | Clustered (mặc định) | Non-clustered (mặc định) |
| Mục đích | Định danh duy nhất cho mỗi hàng | Đảm bảo không trùng lặp |

> **Lưu ý bài test:**
> - Một bảng có thể có **0 hoặc 1** PRIMARY KEY, nhưng **nhiều** UNIQUE
> - UNIQUE cho phép NULL nhưng chỉ **1 hàng NULL** (vì 2 NULL cũng bị coi là trùng trong SQL Server)
> - FOREIGN KEY có thể tham chiếu đến cột PRIMARY KEY hoặc cột UNIQUE của bảng khác
> - CHECK chỉ kiểm tra điều kiện cho **cùng 1 bảng**, không kiểm tra cross-table
