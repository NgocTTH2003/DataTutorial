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

---

## 1. Các câu lệnh SQL quan trọng

| Câu lệnh | Mục đích |
|---|---|
| `SELECT` | Trích xuất dữ liệu từ cơ sở dữ liệu |
| `UPDATE` | Cập nhật dữ liệu trong cơ sở dữ liệu |
| `DELETE` | Xóa dữ liệu khỏi cơ sở dữ liệu |
| `INSERT INTO` | Chèn dữ liệu mới vào cơ sở dữ liệu |
| `CREATE DATABASE` | Tạo cơ sở dữ liệu mới |
| `ALTER DATABASE` | Chỉnh sửa cơ sở dữ liệu |
| `CREATE TABLE` | Tạo một bảng mới |
| `ALTER TABLE` | Chỉnh sửa bảng |
| `DROP TABLE` | Xóa một bảng |
| `CREATE INDEX` | Tạo chỉ mục (khóa tìm kiếm) |
| `DROP INDEX` | Xóa một chỉ mục |

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
