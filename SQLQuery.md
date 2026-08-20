# SQL Server Tutorial

> Tài liệu tổng hợp ôn tập SQL Server / T-SQL — được nhóm theo chủ đề, kèm ví dụ và lưu ý quan trọng.

---

## Mục lục

1. [T-SQL và các quy tắc cơ bản](#1-t-sql-và-các-quy-tắc-cơ-bản)
2. [Kiểu dữ liệu](#2-kiểu-dữ-liệu)
3. [SELECT và lọc dữ liệu](#3-select-và-lọc-dữ-liệu)
4. [DML — Thao tác dữ liệu](#4-dml--thao-tác-dữ-liệu)
5. [DDL — Định nghĩa cấu trúc](#5-ddl--định-nghĩa-cấu-trúc)
6. [Constraints — Ràng buộc](#6-constraints--ràng-buộc)
7. [JOIN](#7-join)
8. [UNION, INTERSECT, EXCEPT](#8-union-intersect-except)
9. [Hàm toán học](#9-hàm-toán-học)
10. [Hàm chuỗi](#10-hàm-chuỗi)
11. [Hàm ngày giờ](#11-hàm-ngày-giờ)
12. [Hàm chuyển đổi kiểu](#12-hàm-chuyển-đổi-kiểu)
13. [Hàm xử lý NULL](#13-hàm-xử-lý-null)
14. [Biểu thức điều kiện](#14-biểu-thức-điều-kiện)
15. [GROUP BY, HAVING và Aggregate Functions](#15-group-by-having-và-aggregate-functions)
16. [Window Functions](#16-window-functions)
17. [Grouping Sets, ROLLUP, CUBE](#17-grouping-sets-rollup-cube)
18. [Subquery — ANY, SOME, ALL](#18-subquery--any-some-all)
19. [CTE và Recursive CTE](#19-cte-và-recursive-cte)
20. [PIVOT và UNPIVOT](#20-pivot-và-unpivot)
21. [Views](#21-views)
22. [Stored Procedures](#22-stored-procedures)
23. [Biến và tham số (DECLARE)](#23-biến-và-tham-số-declare)
24. [Điều khiển luồng (IF, WHILE, BREAK, CONTINUE)](#24-điều-khiển-luồng-if-while-break-continue)
25. [Cursor — Con trỏ](#25-cursor--con-trỏ)
26. [Triggers](#26-triggers)
27. [Xử lý lỗi (TRY-CATCH, RAISERROR, THROW)](#27-xử-lý-lỗi-try-catch-raiserror-throw)
28. [Transaction](#28-transaction)
29. [MERGE](#29-merge)
30. [Dynamic SQL](#30-dynamic-sql)
31. [IDENTITY, GUID, Sequences](#31-identity-guid-sequences)
32. [Index](#32-index)
33. [Partition](#33-partition)
34. [Temporal Tables](#34-temporal-tables)
35. [User-Defined Functions](#35-user-defined-functions)
36. [SYNONYM](#36-synonym)
37. [XML](#37-xml)
38. [Spatial Aggregates](#38-spatial-aggregates)

---

## 1. T-SQL và các quy tắc cơ bản

### T-SQL là gì?

Transact-SQL (T-SQL) là phiên bản mở rộng của SQL do Microsoft phát triển, dùng cho SQL Server và Azure SQL Database. Ngoài các lệnh SQL chuẩn, T-SQL bổ sung thêm biến, vòng lặp, điều kiện, hàm do người dùng định nghĩa, và xử lý lỗi.

### GO — Phân lô câu lệnh

`GO` không phải câu lệnh SQL mà là lệnh của SSMS, dùng để phân tách các batch.

```sql
SELECT 1/1 AS result
GO 3
-- Chạy 3 lần, trả về 3 kết quả
```

### Quy tắc đặt tên

Khi tên chứa khoảng trắng, ký tự đặc biệt, hoặc trùng từ khóa, dùng `[ ]` hoặc `" "`:

```sql
SELECT [Column Name] FROM [My Table];
SELECT "Column Name" FROM "My Table";
```

### Các câu lệnh SQL quan trọng

| Lệnh | Mục đích |
|---|---|
| `SELECT` | Trích xuất dữ liệu |
| `INSERT INTO` | Chèn dữ liệu mới |
| `UPDATE` | Cập nhật dữ liệu |
| `DELETE` | Xóa dữ liệu |
| `CREATE DATABASE / TABLE` | Tạo database / bảng |
| `ALTER DATABASE / TABLE` | Chỉnh sửa database / bảng |
| `DROP TABLE` | Xóa bảng |
| `CREATE INDEX` | Tạo chỉ mục |
| `DROP INDEX` | Xóa chỉ mục |

---

## 2. Kiểu dữ liệu

### 2.1. Kiểu số nguyên

| Kiểu | Kích thước | Phạm vi |
|---|---|---|
| `tinyint` | 1 byte | 0 → 255 |
| `smallint` | 2 bytes | -32,768 → 32,767 |
| `int` | 4 bytes | -2^31 → 2^31-1 |
| `bigint` | 8 bytes | -2^63 → 2^63-1 |

> **Lưu ý:** Nếu kết quả phép tính vượt phạm vi, SQL Server sẽ báo lỗi nhưng vẫn trả về giá trị cũ:
> ```sql
> DECLARE @x AS tinyint = 2
> SET @x = @x - 3  -- Lỗi: vượt phạm vi tinyint
> SELECT @x         -- Vẫn trả về 2
> ```

### 2.2. Kiểu số thực

| Kiểu | Kích thước | Ghi chú |
|---|---|---|
| `decimal(m,d)` / `numeric(m,d)` / `dec(m,d)` | Tùy m | m = tổng chữ số (mặc định 18), d = số thập phân (mặc định 0) |
| `float(n)` | 4-8 bytes | n=1-24: 7 chữ số, 4 bytes. n=25-53: 15 chữ số, 8 bytes |
| `real` | 4 bytes | Tương đương `float(24)` |
| `smallmoney` | 4 bytes | -214,748.3648 → 214,748.3647 |
| `money` | 8 bytes | ±922,337,203,685,477.5807 |

> **Lưu ý quan trọng:** Khi gán số thực vào biến kiểu nguyên, giá trị bị cắt phần thập phân (không làm tròn):
> ```sql
> DECLARE @x AS int = 2
> SET @x = 2.6
> SELECT @x  -- Trả về 2, KHÔNG phải 3
> ```

### 2.3. Kiểu chuỗi

| Kiểu | Bộ ký tự | Kích thước/ký tự | Ghi chú |
|---|---|---|---|
| `char(n)` | ASCII | 1 byte | Độ dài cố định, thêm khoảng trắng nếu ngắn hơn |
| `varchar(n)` | ASCII | 1 byte | Độ dài thay đổi |
| `nchar(n)` | Unicode | 2 bytes | Độ dài cố định |
| `nvarchar(n)` | Unicode | 2 bytes | Độ dài thay đổi |

> **Lưu ý:** `char` luôn chiếm đúng n bytes. Nếu chuỗi ngắn hơn, SQL Server thêm khoảng trắng vào cuối. Dùng `varchar` để tiết kiệm bộ nhớ khi độ dài chuỗi khác nhau nhiều.

### 2.4. Kiểu ngày giờ

Các kiểu chính: `date`, `time`, `datetime`, `datetime2`, `datetimeoffset`, `smalldatetime`

Format chuẩn: `YYYY-MM-DD hh:mm:ss`

```sql
-- Tạo giá trị datetime chi tiết
DATETIMEOFFSETFROMPARTS(year, month, day, hour, minute, seconds, fractions, hour_offset, minute_offset, precision)
```

> **Lưu ý:** Dùng `PARSE()` để chuyển string → datetime, datetime2, decimal.

---

## 3. SELECT và lọc dữ liệu

### 3.1. SELECT cơ bản

```sql
-- Lấy tất cả cột
SELECT * FROM table_name;

-- Lấy cột cụ thể
SELECT col1, col2 FROM table_name;

-- Lấy giá trị không trùng lặp
SELECT DISTINCT col FROM table_name;

-- Đếm giá trị không trùng
SELECT COUNT(DISTINCT Country) FROM Customers;
```

### 3.2. WHERE — Lọc bản ghi

> **Lưu ý:** Giá trị văn bản phải đặt trong dấu nháy đơn `' '`

```sql
SELECT col1, col2 FROM table_name
WHERE condition;
```

Các toán tử: `=` `>` `<` `>=` `<=` `<>` `BETWEEN` `LIKE` `IN`

```sql
-- BETWEEN: lọc trong khoảng
SELECT * FROM Products WHERE Price BETWEEN 50 AND 60;

-- IN: lọc theo danh sách
SELECT * FROM Customers WHERE Country IN ('Germany', 'France');
```

### 3.3. LIKE — Tìm theo mẫu

| Ký tự đại diện | Ý nghĩa | Ví dụ |
|---|---|---|
| `%` | Bất kỳ chuỗi ký tự nào | `LIKE 'A%'` → bắt đầu bằng A |
| `_` | Chính xác 1 ký tự | `LIKE '_at'` → cat, bat |
| `[abc]` | 1 ký tự trong tập hợp | `LIKE '[A-C]%'` → bắt đầu A, B, hoặc C |
| `[^abc]` | 1 ký tự KHÔNG trong tập hợp | `LIKE '[^A-C]%'` → không bắt đầu A, B, C |

### 3.4. AND, OR, NOT — Toán tử logic

```sql
SELECT * FROM Customers WHERE NOT Country = 'Spain';
```

Mở rộng: `NOT LIKE`, `NOT BETWEEN`, `NOT IN`, `IS NOT NULL`, `NOT EXISTS`

### 3.5. ORDER BY — Sắp xếp

```sql
-- ASC = tăng dần (mặc định), DESC = giảm dần
SELECT * FROM Customers
ORDER BY Country ASC, CustomerName DESC;
```

---

## 4. DML — Thao tác dữ liệu

### 4.1. INSERT INTO

```sql
-- Cú pháp 1: Chỉ định cột
INSERT INTO table_name (col1, col2, col3)
VALUES (val1, val2, val3);

-- Cú pháp 2: Tất cả cột (theo đúng thứ tự)
INSERT INTO table_name
VALUES (val1, val2, val3);
```

**Chèn nhiều hàng:**

```sql
INSERT INTO sales.promotions (promotion_name, discount, start_date, expired_date)
VALUES
    ('2020 Summer Promotion', 0.25, '20200601', '20200901'),
    ('2020 Fall Promotion', 0.10, '20201001', '20201101'),
    ('2020 Winter Promotion', 0.25, '20201201', '20210101');
```

**Chèn vào cột IDENTITY:**

```sql
SET IDENTITY_INSERT sales.promotions ON;

INSERT INTO sales.promotions (promotion_id, promotion_name, discount, start_date, expired_date)
VALUES (4, '2019 Spring Promotion', 0.25, '20190201', '20190301');

SET IDENTITY_INSERT sales.promotions OFF;
```

> **Lưu ý:** Cột IDENTITY tự tăng, mặc định không cho chèn thủ công. Phải bật `IDENTITY_INSERT` trước.

### 4.2. INSERT INTO SELECT

> **Yêu cầu:** Bảng đích phải tồn tại trước.

```sql
-- Chèn tất cả
INSERT INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers;

-- Chèn top N dòng
INSERT TOP (10) INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers;

-- Chèn top N phần trăm
INSERT TOP (10) PERCENT INTO sales.addresses (street, city, state, zip_code)
SELECT street, city, state, zip_code
FROM sales.customers;
```

### 4.3. UPDATE

**Cách 1: UPDATE cơ bản**

```sql
UPDATE table_name
SET col1 = val1, col2 = val2
WHERE condition;
```

**Cách 2: UPDATE với JOIN**

```sql
-- Cập nhật bảng A dựa trên dữ liệu từ bảng B
UPDATE a
SET a.price = b.new_price
FROM products a
INNER JOIN price_updates b ON a.id = b.product_id;
```

**Cách 3: UPDATE với Subquery**

```sql
UPDATE products
SET price = (SELECT AVG(price) FROM products WHERE category = 'A')
WHERE category = 'B';
```

**Cách 4: UPDATE với CTE**

```sql
WITH cte AS (
    SELECT id, price, ROW_NUMBER() OVER(PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
)
UPDATE cte
SET price = price * 0.9
WHERE rn = 1;  -- Giảm 10% sản phẩm đắt nhất mỗi loại
```

**Cách 5: UPDATE TOP**

```sql
-- Cập nhật N dòng đầu tiên
UPDATE TOP (10) products
SET price = price * 1.1
WHERE category = 'A';
```

**Cách 6: UPDATE với OUTPUT (xem giá trị trước/sau)**

```sql
UPDATE products
SET price = price * 1.1
OUTPUT deleted.price AS gia_cu, inserted.price AS gia_moi
WHERE category = 'A';
```

### 4.4. DELETE, TRUNCATE, DROP — So sánh 3 cách xóa

| Tiêu chí | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Xóa gì | Dữ liệu (có WHERE) | Toàn bộ dữ liệu | Cả bảng + cấu trúc |
| Giữ cấu trúc bảng | Có | Có | Không |
| Dùng WHERE | Có | Không | Không |
| Kích hoạt Trigger | Có | Không | Không |
| Tốc độ | Chậm (xóa từng hàng) | Nhanh | Nhanh |
| Khóa ngoại | Được (nếu có CASCADE) | Không được nếu có FK tham chiếu | Không được nếu có FK |
| Rollback | Có | Có | Có |

```sql
DELETE FROM table_name WHERE condition;
TRUNCATE TABLE table_name;
DROP TABLE table_name;
```

### 4.5. OUTPUT — "Biên nhận" sau thao tác

> **OUTPUT** giống hóa đơn: câu lệnh vẫn chạy bình thường nếu không có, nhưng có thì SQL Server tự trả kết quả luôn mà không cần `SELECT` lại.

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

## 5. DDL — Định nghĩa cấu trúc

### ALTER TABLE

```sql
-- Thêm cột
ALTER TABLE table_name ADD col_name datatype;

-- Xóa cột
ALTER TABLE table_name DROP COLUMN col_name;

-- Thay đổi kiểu dữ liệu
ALTER TABLE table_name ALTER COLUMN col_name new_datatype;

-- Thêm ràng buộc
ALTER TABLE table_name ADD CONSTRAINT constraint_name ...;

-- Xóa ràng buộc
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```

---

## 6. Constraints — Ràng buộc

### Các loại ràng buộc

```sql
-- PRIMARY KEY
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (EmployeeID);

-- FOREIGN KEY
ALTER TABLE orders
ADD CONSTRAINT fk_employee FOREIGN KEY (EmployeeID) REFERENCES employees(EmployeeID);

-- UNIQUE
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (Email);

-- CHECK
ALTER TABLE employees
ADD CONSTRAINT check_age CHECK (Age >= 18);

-- DEFAULT
ALTER TABLE employees
ADD CONSTRAINT default_start_date DEFAULT GETDATE() FOR StartDate;

-- NOT NULL (dùng ALTER COLUMN)
ALTER TABLE employees ALTER COLUMN Name VARCHAR(100) NOT NULL;

-- Xóa ràng buộc
ALTER TABLE employees DROP CONSTRAINT pk_employee_id;
```

### PRIMARY KEY vs UNIQUE

| Tiêu chí | PRIMARY KEY | UNIQUE |
|---|---|---|
| Số lượng trên bảng | Chỉ 1 | Nhiều |
| Cho phép NULL | Không | Có (tối đa 1 hàng NULL) |
| Tạo Index | Clustered (mặc định) | Non-clustered |

> **Lưu ý:** UNIQUE có thể có 0 hoặc 1 hàng NULL, không được nhiều hơn.

---

## 7. JOIN

```sql
-- INNER JOIN: chỉ lấy dòng khớp ở cả 2 bảng
SELECT * FROM A INNER JOIN B ON A.id = B.id;

-- LEFT JOIN: lấy tất cả từ bảng trái, khớp từ bảng phải
SELECT * FROM A LEFT JOIN B ON A.id = B.id;

-- RIGHT JOIN: lấy tất cả từ bảng phải, khớp từ bảng trái
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;

-- FULL JOIN: lấy tất cả từ cả 2 bảng
SELECT * FROM A FULL JOIN B ON A.id = B.id;

-- CROSS JOIN: tích Descartes (mỗi dòng A ghép với mọi dòng B)
SELECT * FROM A CROSS JOIN B;

-- SELF JOIN: bảng join với chính nó
SELECT a.col, b.col FROM table_name a, table_name b
WHERE condition;
```

---

## 8. UNION, INTERSECT, EXCEPT

### UNION / UNION ALL

```sql
-- UNION: hợp nhất kết quả, loại bỏ trùng
SELECT col FROM A UNION SELECT col FROM B;

-- UNION ALL: hợp nhất kết quả, giữ cả trùng
SELECT col FROM A UNION ALL SELECT col FROM B;
```

> **Lưu ý:** UNION khác JOIN — UNION ghép hàng theo chiều dọc (cùng cấu trúc cột), JOIN ghép cột theo chiều ngang.

### INTERSECT — Giao

```sql
-- Trả về các hàng có ở CẢ HAI truy vấn
SELECT col FROM A INTERSECT SELECT col FROM B;
```

### EXCEPT — Hiệu (phép trừ)

```sql
-- Trả về các hàng có ở truy vấn 1 nhưng KHÔNG có ở truy vấn 2
SELECT col FROM A EXCEPT SELECT col FROM B;
```

---

## 9. Hàm toán học

| Hàm | Ý nghĩa | Ví dụ |
|---|---|---|
| `ABS(x)` | Giá trị tuyệt đối | `ABS(-5)` → 5 |
| `SIGN(x)` | Dấu của x (-1, 0, 1) | `SIGN(-5)` → -1 |
| `POWER(x,y)` | x mũ y | `POWER(2,3)` → 8 |
| `SQUARE(x)` | x bình phương | `SQUARE(4)` → 16 |
| `SQRT(x)` | Căn bậc 2 | `SQRT(16)` → 4 |
| `FLOOR(x)` | Làm tròn xuống | `FLOOR(5.7)` → 5, `FLOOR(-5.7)` → -6 |
| `CEILING(x)` | Làm tròn lên | `CEILING(5.7)` → 6, `CEILING(-5.7)` → -5 |
| `ROUND(x,y)` | Làm tròn y chữ số | `ROUND(123.43, -1)` → 120 |
| `PI()` | Giá trị số Pi | 3.14159... |
| `RAND()` | Số ngẫu nhiên 0-1 | |

Danh sách đầy đủ: `ABS, ACOS, ASIN, ATAN, ATN2, CEILING, COS, COT, DEGREES, EXP, FLOOR, LOG, LOG10, PI, POWER, RADIANS, RAND, ROUND, SIGN, SIN, SQRT, SQUARE, TAN`

---

## 10. Hàm chuỗi

### Chuyển đổi ký tự

| Hàm | Ý nghĩa |
|---|---|
| `ASCII(char)` | Mã ASCII của ký tự |
| `CHAR(int)` | Ký tự từ mã ASCII |
| `NCHAR(int)` | Ký tự Unicode từ mã |
| `UNICODE(char)` | Mã Unicode của ký tự |

### Vị trí và tìm kiếm

| Hàm | Ý nghĩa |
|---|---|
| `CHARINDEX(substr, str)` | Vị trí xuất hiện đầu tiên |
| `PATINDEX('%pattern%', str)` | Vị trí theo mẫu pattern |

### Nối chuỗi

| Hàm | Ý nghĩa |
|---|---|
| `CONCAT(s1, s2, ...)` | Nối nhiều chuỗi |
| `CONCAT_WS(sep, s1, s2, ...)` | Nối chuỗi với ký tự phân tách |
| `STRING_AGG(expr, sep)` | Gộp nhiều hàng thành 1 chuỗi |

### Thao tác chuỗi con

| Hàm | Ý nghĩa |
|---|---|
| `LEFT(str, n)` | Lấy n ký tự từ đầu |
| `RIGHT(str, n)` | Lấy n ký tự từ cuối |
| `SUBSTRING(str, start, len)` | Trích xuất chuỗi con |
| `STUFF(str, start, len, new)` | Xóa + chèn chuỗi mới |

### Định dạng và thay thế

| Hàm | Ý nghĩa |
|---|---|
| `FORMAT(value, format)` | Định dạng số/ngày |
| `REPLACE(str, old, new)` | Thay thế chuỗi con |
| `TRANSLATE(str, from, to)` | Thay thế từng ký tự |

### Khoảng trắng và khác

| Hàm | Ý nghĩa |
|---|---|
| `LTRIM / RTRIM / TRIM` | Xóa khoảng trắng trái/phải/cả hai |
| `LEN(str)` | Số ký tự (không tính space cuối) |
| `LOWER / UPPER` | Chữ thường / chữ hoa |
| `REVERSE(str)` | Đảo ngược chuỗi |
| `SPACE(n)` | Tạo n khoảng trắng |
| `STR(value, len, dec)` | Chuyển số thành chuỗi |
| `STRING_SPLIT(str, sep)` | Tách chuỗi thành danh sách |
| `QUOTENAME(str)` | Thêm `[ ]` hoặc `" "` quanh chuỗi |
| `SOUNDEX(str)` | Mã phát âm |
| `DIFFERENCE(s1, s2)` | Độ giống nhau về phát âm (0-4) |

---

## 11. Hàm ngày giờ

```sql
-- Lấy ngày hiện tại
SELECT GETDATE(), SYSDATETIME(), CURRENT_TIMESTAMP;

-- Trích xuất phần ngày
SELECT YEAR(date), MONTH(date), DAY(date);
SELECT DATEPART(weekday, date), DATENAME(month, date);

-- Tính toán ngày
SELECT DATEADD(day, 7, GETDATE());       -- Cộng 7 ngày
SELECT DATEDIFF(day, start_date, end_date); -- Số ngày chênh lệch

-- Chuyển đổi string → datetime
SELECT PARSE('01/01/2024' AS date USING 'vi-VN');
```

---

## 12. Hàm chuyển đổi kiểu

### CAST vs CONVERT

| Tiêu chí | CAST | CONVERT |
|---|---|---|
| Chuẩn | SQL ANSI (tương thích cao) | Mở rộng của SQL Server |
| Định dạng | Không hỗ trợ style | Có hỗ trợ style (đặc biệt ngày giờ) |

```sql
-- CAST
SELECT CAST(123.45 AS int);            -- 123
SELECT CAST('2024-01-01' AS date);

-- CONVERT (có style cho ngày)
SELECT CONVERT(varchar, GETDATE(), 103);  -- dd/mm/yyyy
SELECT CONVERT(varchar, GETDATE(), 120);  -- yyyy-mm-dd hh:mm:ss
```

### TRY_CONVERT

Giống CONVERT nhưng trả về NULL thay vì lỗi khi chuyển đổi thất bại:

```sql
SELECT TRY_CONVERT(int, 'abc');  -- NULL, không lỗi
```

### FORMAT

```sql
SELECT FORMAT(12345.6789, 'N2');            -- 12,345.68
SELECT FORMAT(GETDATE(), 'dd/MM/yyyy');     -- 20/08/2026
SELECT FORMAT(12345, 'C', 'vi-VN');         -- ₫12.345
```

---

## 13. Hàm xử lý NULL

### COALESCE

Trả về giá trị không NULL đầu tiên:

```sql
SELECT COALESCE(NULL, NULL, 'Hello', 'World');  -- 'Hello'
SELECT COALESCE(phone, email, 'N/A') FROM customers;
```

### ISNULL

Tương tự COALESCE nhưng chỉ nhận 2 tham số:

```sql
SELECT ISNULL(phone, 'Chưa có SĐT') FROM customers;
```

| Tiêu chí | COALESCE | ISNULL |
|---|---|---|
| Số tham số | Nhiều | Chỉ 2 |
| Chuẩn SQL | Có (ANSI) | Không (chỉ SQL Server) |
| Kiểu trả về | Kiểu có độ ưu tiên cao nhất | Kiểu của tham số đầu |

### NULLIF

Trả về NULL nếu 2 giá trị bằng nhau, ngược lại trả về giá trị đầu:

```sql
SELECT NULLIF(10, 10);  -- NULL
SELECT NULLIF(10, 20);  -- 10
```

---

## 14. Biểu thức điều kiện

### IIF

```sql
SELECT IIF(Price > 100, 'Đắt', 'Rẻ') AS DanhGia FROM Products;
```

### CASE

```sql
-- Dạng 1: Simple CASE
SELECT
    CASE status
        WHEN 'A' THEN 'Active'
        WHEN 'I' THEN 'Inactive'
        ELSE 'Unknown'
    END AS TrangThai
FROM table_name;

-- Dạng 2: Searched CASE
SELECT
    CASE
        WHEN price > 100 THEN 'Đắt'
        WHEN price > 50 THEN 'Vừa'
        ELSE 'Rẻ'
    END AS DanhGia
FROM Products;
```

---

## 15. GROUP BY, HAVING và Aggregate Functions

### GROUP BY

```sql
SELECT Country, COUNT(*) AS SoLuong
FROM Customers
GROUP BY Country;
```

### HAVING — Lọc sau khi nhóm

> **Lưu ý:** `WHERE` lọc trước GROUP BY, `HAVING` lọc sau GROUP BY. HAVING dùng được với hàm tổng hợp, WHERE thì không.

```sql
SELECT Country, COUNT(*) AS SoLuong
FROM Customers
GROUP BY Country
HAVING COUNT(*) > 5;
```

### Các hàm tổng hợp

`COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`

---

## 16. Window Functions

### OVER() — Tính toán không cần GROUP BY

`OVER()` giữ nguyên các dòng gốc, không gộp lại như GROUP BY:

```sql
SELECT
    name,
    salary,
    SUM(salary) OVER() AS tong_luong,
    SUM(salary) OVER(PARTITION BY department) AS tong_theo_phong
FROM employees;
```

### PARTITION BY — Chia nhóm trong OVER

```sql
SELECT
    name, department, salary,
    AVG(salary) OVER(PARTITION BY department) AS avg_phong
FROM employees;
```

### ORDER BY trong OVER

```sql
SELECT
    name, salary,
    SUM(salary) OVER(ORDER BY salary) AS running_total
FROM employees;
```

### ROWS BETWEEN — Khung cửa sổ

```sql
-- 2 hàng trước + hàng hiện tại + 2 hàng sau
SUM(salary) OVER(ORDER BY id ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING)

-- Từ đầu đến hàng hiện tại
SUM(salary) OVER(ORDER BY id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- Từ hàng hiện tại đến cuối
SUM(salary) OVER(ORDER BY id ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING)
```

> **Lưu ý:** `CURRENT ROW` tương đương `0 PRECEDING`

### ROWS vs RANGE

| Tiêu chí | ROWS | RANGE |
|---|---|---|
| Cơ sở | Vị trí vật lý (số hàng) | Giá trị logic (ORDER BY) |
| Khi trùng giá trị | Xử lý từng hàng riêng | Gộp các hàng trùng lại |
| Hiệu suất | Tốt hơn | Kém hơn |
| Khuyến nghị | Thường dùng hơn | Cần khi ORDER BY có trùng |

### Hàm xếp hạng

```sql
SELECT
    name, salary,
    ROW_NUMBER() OVER(ORDER BY salary DESC) AS stt,          -- 1, 2, 3, 4
    RANK()       OVER(ORDER BY salary DESC) AS rank_skip,    -- 1, 2, 2, 4 (bỏ qua)
    DENSE_RANK() OVER(ORDER BY salary DESC) AS rank_dense,   -- 1, 2, 2, 3 (liên tục)
    NTILE(4)     OVER(ORDER BY salary DESC) AS quartile      -- Chia 4 nhóm
FROM employees;
```

### FIRST_VALUE, LAST_VALUE

```sql
SELECT
    name, salary,
    FIRST_VALUE(name) OVER(ORDER BY salary DESC) AS nguoi_luong_cao_nhat,
    LAST_VALUE(name) OVER(ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS nguoi_luong_thap_nhat
FROM employees;
```

> **Lưu ý:** `LAST_VALUE` mặc định chỉ tính đến hàng hiện tại. Phải thêm `ROWS BETWEEN ... UNBOUNDED FOLLOWING` để lấy đúng giá trị cuối cùng.

### LAG, LEAD

```sql
SELECT
    name, salary,
    LAG(salary, 1)  OVER(ORDER BY id) AS luong_truoc,  -- Giá trị hàng trước
    LEAD(salary, 1) OVER(ORDER BY id) AS luong_sau     -- Giá trị hàng sau
FROM employees;
```

### CUME_DIST, PERCENT_RANK

```sql
SELECT
    name, salary,
    CUME_DIST()    OVER(ORDER BY salary) AS cum_dist,      -- Phân phối tích lũy
    PERCENT_RANK() OVER(ORDER BY salary) AS percent_rank   -- Xếp hạng phần trăm
FROM employees;
```

### PERCENTILE_CONT, PERCENTILE_DISC

```sql
-- PERCENTILE_CONT: nội suy (có thể trả về giá trị không có trong data)
SELECT DISTINCT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary)
        OVER(PARTITION BY department) AS median_cont
FROM employees;

-- PERCENTILE_DISC: lấy giá trị thực tế gần nhất
SELECT DISTINCT
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary)
        OVER(PARTITION BY department) AS median_disc
FROM employees;
```

---

## 17. Grouping Sets, ROLLUP, CUBE

### ROLLUP — Tổng hợp theo phân cấp

`ROLLUP(d1, d2, d3)` tạo 4 nhóm: (d1,d2,d3), (d1,d2), (d1), ()

```sql
SELECT brand, category, SUM(sales) AS total
FROM products
GROUP BY ROLLUP(brand, category);
```

### CUBE — Tất cả tổ hợp

`CUBE(d1, d2)` tạo mọi tổ hợp: (d1,d2), (d1), (d2), ()

```sql
SELECT brand, category, SUM(sales) AS total
FROM products
GROUP BY CUBE(brand, category);
```

### GROUPING SETS — Tùy chỉnh nhóm

```sql
SELECT brand, category, SUM(sales) AS total
FROM products
GROUP BY GROUPING SETS (
    (brand, category),
    (brand),
    ()
);
```

### GROUPING và GROUPING_ID

```sql
-- GROUPING(col): 1 = đang tổng hợp (NULL do gộp), 0 = không
SELECT
    brand, category,
    SUM(sales) AS total,
    GROUPING(brand) AS is_brand_total,
    GROUPING_ID(brand, category) AS group_level
FROM products
GROUP BY ROLLUP(brand, category);
```

> **Lưu ý:** `GROUPING(brand)` = 1 nghĩa là brand đang ở mức tổng hợp (NULL do ROLLUP/CUBE tạo ra, không phải NULL thật).

---

## 18. Subquery — ANY, SOME, ALL

```sql
-- ANY / SOME: trả về TRUE nếu bất kỳ giá trị nào thỏa mãn
SELECT * FROM Products
WHERE Price > ANY (SELECT Price FROM Products WHERE Category = 'A');

-- ALL: trả về TRUE nếu TẤT CẢ giá trị thỏa mãn
SELECT * FROM Products
WHERE Price > ALL (SELECT Price FROM Products WHERE Category = 'A');
```

> **Lưu ý:** `ANY` và `SOME` hoàn toàn giống nhau.

---

## 19. CTE và Recursive CTE

### CTE — Common Table Expression

```sql
WITH cte_name AS (
    SELECT col1, col2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name;
```

> **Lưu ý:** CTE là truy vấn tạm, chỉ tồn tại trong phạm vi câu lệnh ngay sau `WITH`. Có thể tham chiếu nhiều lần trong cùng câu lệnh.

### Recursive CTE — Đệ quy

Cấu trúc gồm: anchor member (truy vấn ban đầu) + recursive member (tham chiếu chính nó) + UNION ALL

```sql
-- Ví dụ: sơ đồ tổ chức
WITH org_chart AS (
    -- Anchor: cấp cao nhất
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: tìm nhân viên cấp dưới
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart;
```

> **Lưu ý:** Mặc định SQL Server giới hạn 100 level đệ quy. Thêm `OPTION (MAXRECURSION n)` để thay đổi.

---

## 20. PIVOT và UNPIVOT

### PIVOT — Hàng → Cột

```sql
SELECT *
FROM (
    SELECT year, category, revenue
    FROM sales
) AS source
PIVOT (
    SUM(revenue)
    FOR category IN ([Electronics], [Clothing], [Food])
) AS pvt;
```

### UNPIVOT — Cột → Hàng

```sql
SELECT year, category, revenue
FROM sales_pivot
UNPIVOT (
    revenue FOR category IN ([Electronics], [Clothing], [Food])
) AS unpvt;
```

---

## 21. Views

### Tạo, sửa, xóa View

```sql
-- Tạo
CREATE VIEW vw_active_employees AS
SELECT EmployeeID, Name, Salary
FROM employees
WHERE Status = 'Active';

-- Sửa
ALTER VIEW vw_active_employees AS
SELECT EmployeeID, Name, Salary, Department
FROM employees
WHERE Status = 'Active';

-- Xóa
DROP VIEW vw_active_employees;
```

### Các thuộc tính đặc biệt

**SCHEMABINDING** — Bảo vệ cấu trúc: không cho phép ALTER/DROP bảng gốc khi View đang dùng.

```sql
ALTER VIEW dbo.vw_employees
WITH SCHEMABINDING AS
SELECT EmployeeID, Name, Salary
FROM dbo.employees;
```

> **Lưu ý:** Phải chỉ định tên schema (dbo.) và liệt kê rõ từng cột (không dùng `*`).

**ENCRYPTION** — Mã hóa định nghĩa View:

```sql
ALTER VIEW vw_employees WITH ENCRYPTION AS ...;
```

**WITH CHECK OPTION** — Đảm bảo INSERT/UPDATE qua View vẫn thỏa mãn điều kiện WHERE:

```sql
ALTER VIEW vw_active AS
SELECT * FROM employees WHERE Status = 'Active'
WITH CHECK OPTION;
-- INSERT qua View mà Status <> 'Active' sẽ bị từ chối
```

### Lưu ý quan trọng về View

- Trong View chỉ dùng được `ORDER BY` khi kết hợp với `TOP`
- Có thể INSERT, UPDATE, DELETE qua View (nếu View đơn giản, 1 bảng)
- Có thể tạo Index trên View (cải thiện hiệu suất, bắt buộc có SCHEMABINDING)
- SQL Server không hỗ trợ đổi tên View — phải DROP rồi CREATE lại
- Dùng `ALTER MATERIALIZED VIEW ... REFRESH` để làm mới dữ liệu

---

## 22. Stored Procedures

### Tạo, chạy, sửa, xóa

```sql
-- Tạo
CREATE PROCEDURE sp_get_customers
AS
BEGIN
    SELECT * FROM customers WHERE status = 'Active';
END;

-- Chạy
EXEC sp_get_customers;

-- Sửa
ALTER PROCEDURE sp_get_customers AS ...;

-- Xóa
DROP PROCEDURE sp_get_customers;
```

### Tham số đầu vào

```sql
CREATE PROCEDURE sp_get_by_city
    @city NVARCHAR(50),
    @status NVARCHAR(20) = 'Active'  -- Giá trị mặc định
AS
BEGIN
    SELECT * FROM customers
    WHERE city = @city AND status = @status;
END;

-- Gọi
EXEC sp_get_by_city @city = N'Hà Nội';
EXEC sp_get_by_city @city = N'Hà Nội', @status = 'VIP';
```

### Tham số đầu ra (OUTPUT)

```sql
CREATE PROCEDURE sp_count_customers
    @city NVARCHAR(50),
    @total INT OUTPUT
AS
BEGIN
    SELECT @total = COUNT(*)
    FROM customers
    WHERE city = @city;
END;

-- Gọi
DECLARE @result INT;
EXEC sp_count_customers @city = N'Hà Nội', @total = @result OUTPUT;
SELECT @result;
```

> **Lưu ý:** Stored Procedure được biên dịch sẵn, lần gọi sau không cần biên dịch lại → tối ưu hiệu suất.

---

## 23. Biến và tham số (DECLARE)

### Khai báo và sử dụng biến

```sql
-- Khai báo
DECLARE @name NVARCHAR(50);
DECLARE @age INT = 25;

-- Gán giá trị
SET @name = N'Nguyễn Văn A';

-- Sử dụng trong truy vấn
SELECT * FROM customers WHERE name = @name AND age > @age;
```

> **Lưu ý:**
> - Biến bắt đầu bằng `@`
> - Phạm vi: chỉ tồn tại trong batch hoặc stored procedure hiện tại
> - Mỗi biến khác kiểu dữ liệu phải khai báo riêng

---

## 24. Điều khiển luồng (IF, WHILE, BREAK, CONTINUE)

### IF

```sql
IF @quantity > 0
BEGIN
    PRINT 'Còn hàng';
END
ELSE
BEGIN
    PRINT 'Hết hàng';
END;
```

### WHILE

```sql
DECLARE @i INT = 1;
WHILE @i <= 10
BEGIN
    PRINT @i;
    SET @i = @i + 1;
END;
```

### BREAK và CONTINUE

```sql
DECLARE @i INT = 0;
WHILE @i < 100
BEGIN
    SET @i = @i + 1;
    IF @i = 5 CONTINUE;    -- Bỏ qua, chạy vòng tiếp
    IF @i = 10 BREAK;       -- Thoát vòng lặp
    PRINT @i;
END;
```

---

## 25. Cursor — Con trỏ

Cursor xử lý dữ liệu theo từng dòng (row-by-row) thay vì toàn bộ tập dữ liệu.

### Khi nào dùng

- Gửi email cho từng khách hàng
- Xử lý logic phức tạp cần duyệt từng hàng
- Tương tác với dữ liệu tạm mà không thay đổi bảng gốc

> **Lưu ý:** KHÔNG nên dùng Cursor với data lớn hoặc batch processing — rất chậm. Ưu tiên dùng set-based operations (JOIN, CTE).

### Các bước sử dụng

```sql
-- 1. Khai báo
DECLARE cursor_name CURSOR FOR
    SELECT col1, col2 FROM table_name;

-- 2. Mở
OPEN cursor_name;

-- 3. Duyệt từng hàng
DECLARE @col1 INT, @col2 NVARCHAR(50);
FETCH NEXT FROM cursor_name INTO @col1, @col2;

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Xử lý từng hàng
    PRINT @col2;
    FETCH NEXT FROM cursor_name INTO @col1, @col2;
END;

-- 4. Đóng
CLOSE cursor_name;

-- 5. Hủy
DEALLOCATE cursor_name;
```

> **Lưu ý:** `@@FETCH_STATUS = 0` nghĩa là FETCH thành công. Khác 0 là hết dữ liệu.

---

## 26. Triggers

### Trigger là gì?

- Là stored procedure đặc biệt, **tự động thực thi** khi có INSERT/UPDATE/DELETE
- Phải liên kết với table/view
- Không thể gọi thủ công, không có tham số

### Hai loại Trigger chính

| Loại | Kích hoạt bởi | Phạm vi |
|---|---|---|
| **DML Trigger** | INSERT, UPDATE, DELETE | Table / View |
| **DDL Trigger** | CREATE, ALTER, DROP | Database / Schema |

### AFTER Trigger vs INSTEAD OF Trigger

| Loại | Thực hiện khi |
|---|---|
| `AFTER` | Sau khi thao tác DML hoàn tất |
| `INSTEAD OF` | Thay thế thao tác DML (thao tác gốc KHÔNG được thực hiện) |

### Tạo Trigger

```sql
-- AFTER Trigger: ghi log sau khi INSERT
CREATE TRIGGER trg_after_insert
ON employees
AFTER INSERT
AS
BEGIN
    INSERT INTO audit_log (action, table_name, timestamp)
    SELECT 'INSERT', 'employees', GETDATE()
    FROM inserted;
END;
```

```sql
-- INSTEAD OF Trigger: kiểm tra trước khi DELETE
CREATE TRIGGER trg_instead_delete
ON employees
INSTEAD OF DELETE
AS
BEGIN
    -- Chỉ cho phép xóa nhân viên đã nghỉ
    DELETE FROM employees
    WHERE id IN (SELECT id FROM deleted WHERE status = 'Resigned');
END;
```

### Bảng tạm inserted và deleted

| Bảng | Chứa dữ liệu |
|---|---|
| `inserted` | Dòng mới (INSERT) hoặc giá trị mới (UPDATE) |
| `deleted` | Dòng bị xóa (DELETE) hoặc giá trị cũ (UPDATE) |

### Nested Trigger — Trigger lồng nhau

Trigger có thể kích hoạt trigger khác. Mức lồng tối đa: 32. Dùng `@@NESTLEVEL` để kiểm tra mức hiện tại.

---

## 27. Xử lý lỗi (TRY-CATCH, RAISERROR, THROW)

### TRY-CATCH

```sql
BEGIN TRY
    -- Code có thể gây lỗi
    SELECT 1/0;
END TRY
BEGIN CATCH
    SELECT
        ERROR_NUMBER()    AS ErrorNumber,
        ERROR_MESSAGE()   AS ErrorMessage,
        ERROR_SEVERITY()  AS Severity,
        ERROR_STATE()     AS State,
        ERROR_LINE()      AS Line,
        ERROR_PROCEDURE() AS ProcedureName;
END CATCH;
```

> **Lưu ý:**
> - Nếu TRY không lỗi thì CATCH không chạy
> - Các hàm `ERROR_*()` chỉ hoạt động trong khối CATCH
> - Có thể lồng TRY-CATCH bên trong nhau

### RAISERROR — Tạo lỗi tùy chỉnh

```sql
RAISERROR('Giá trị không hợp lệ', 16, 1);
-- severity: 0-25 (16 = user error)
-- state: 0-255
```

> **Lưu ý:** Message code do người dùng định nghĩa phải > 50000. Dùng `sp_addmessage` để thêm.

### THROW — Ném ngoại lệ

```sql
THROW 50001, N'Không tìm thấy dữ liệu', 1;
-- error_number: > 50000 và <= 2,147,483,647
```

Dùng `THROW` không tham số trong CATCH để re-throw lỗi gốc:

```sql
BEGIN CATCH
    PRINT 'Đã xảy ra lỗi';
    THROW;  -- Ném lại lỗi ban đầu
END CATCH;
```

### RAISERROR vs THROW

| Tiêu chí | RAISERROR | THROW |
|---|---|---|
| Phiên bản | Cũ hơn | SQL Server 2012+ |
| Kết thúc bằng `;` | Không bắt buộc | Bắt buộc câu trước phải có `;` |
| Re-throw lỗi | Không hỗ trợ | Hỗ trợ (THROW không tham số) |
| Severity | Tùy chỉnh 0-25 | Luôn là 16 |
| Khuyến nghị | Legacy code | Dùng cho code mới |

---

## 28. Transaction

```sql
BEGIN TRANSACTION;
    DELETE TOP(5) FROM orders;
    -- Nếu sai, rollback
ROLLBACK TRANSACTION;

-- Hoặc commit nếu đúng
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT TRANSACTION;
```

---

## 29. MERGE

Kết hợp INSERT, UPDATE, DELETE trong 1 câu lệnh dựa trên bảng nguồn và bảng đích:

```sql
MERGE INTO target_table AS t
USING source_table AS s
ON t.id = s.id
WHEN MATCHED THEN
    UPDATE SET t.name = s.name, t.price = s.price
WHEN NOT MATCHED BY TARGET THEN
    INSERT (id, name, price) VALUES (s.id, s.name, s.price)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

> **Lưu ý:** MERGE giải quyết 3 trường hợp cùng lúc:
> - Có ở nguồn, không có ở đích → INSERT
> - Có ở cả hai nhưng khác giá trị → UPDATE
> - Có ở đích, không có ở nguồn → DELETE

---

## 30. Dynamic SQL

Tạo và thực thi câu lệnh SQL tại runtime:

```sql
DECLARE @table NVARCHAR(50) = 'employees';
DECLARE @sql NVARCHAR(MAX);

SET @sql = N'SELECT * FROM ' + QUOTENAME(@table);
EXEC sp_executesql @sql;
```

> **Lưu ý:** Luôn dùng `QUOTENAME()` hoặc tham số hóa để tránh SQL Injection.

---

## 31. IDENTITY, GUID, Sequences

### IDENTITY — Tự tăng

```sql
CREATE TABLE orders (
    id INT IDENTITY(1,1) PRIMARY KEY,  -- seed=1, increment=1
    name NVARCHAR(100)
);
```

### GUID — Giá trị duy nhất toàn cầu

```sql
DECLARE @id UNIQUEIDENTIFIER = NEWID();
SELECT @id;  -- VD: 6F9619FF-8B86-D011-B42D-00CF4FC964FF
```

> **Lưu ý:** GUID thường dùng trong hệ thống phân tán vì đảm bảo duy nhất mà không cần đồng bộ giữa các server.

### Sequences — Đối tượng tạo số tự tăng độc lập

```sql
-- Tạo sequence
CREATE SEQUENCE seq_order_id
    START WITH 1
    INCREMENT BY 1;

-- Sử dụng
INSERT INTO orders (id, name)
VALUES (NEXT VALUE FOR seq_order_id, N'Order A');
```

| Tiêu chí | IDENTITY | Sequence |
|---|---|---|
| Phạm vi | 1 cột, 1 bảng | Dùng được nhiều bảng |
| Kiểm soát | Hạn chế | Linh hoạt (reset, skip) |
| Lấy giá trị trước INSERT | Không | Có (`NEXT VALUE FOR`) |

---

## 32. Index

### Clustered Index

- Dữ liệu được sắp xếp vật lý theo thứ tự index
- Mỗi bảng chỉ có **1** clustered index
- Mặc định PRIMARY KEY tạo clustered index

### Non-Clustered Index

- Lưu trữ tham chiếu (con trỏ) đến dữ liệu thực
- Mỗi bảng có thể có **nhiều** non-clustered index
- Tối ưu truy vấn tìm kiếm trên cột không phải PK

```sql
-- Tạo non-clustered index
CREATE NONCLUSTERED INDEX idx_customer_name
ON customers (name);

-- Tạo clustered index
CREATE CLUSTERED INDEX idx_order_date
ON orders (order_date);
```

| Tiêu chí | Clustered | Non-Clustered |
|---|---|---|
| Sắp xếp dữ liệu vật lý | Có | Không |
| Số lượng / bảng | 1 | Nhiều |
| Tốc độ đọc | Nhanh hơn (range scan) | Nhanh cho tìm kiếm cụ thể |
| Tốc độ ghi | Chậm hơn (phải sắp xếp lại) | Nhanh hơn |

> **Lưu ý:** Nếu không có index, mỗi truy vấn phải quét toàn bộ bảng (table scan). Index tạo cấu trúc dữ liệu giúp xác định nhanh vị trí dòng cần tìm.

---

## 33. Partition

Chia bảng lớn thành nhiều phần nhỏ để tối ưu hiệu suất.

### Lợi ích

- Truy vấn chỉ quét partition liên quan thay vì toàn bộ bảng
- Xóa data theo partition nhanh hơn xóa từng dòng
- Index trên partition nhỏ hơn, bảo trì dễ hơn

### Các bước tạo Partition

```sql
-- 1. Tạo Partition Function (chia dữ liệu theo điều kiện)
CREATE PARTITION FUNCTION pf_year (INT)
AS RANGE LEFT FOR VALUES (2022, 2023, 2024);

-- 2. Tạo Partition Scheme (ánh xạ partition → filegroup)
CREATE PARTITION SCHEME ps_year
AS PARTITION pf_year TO (fg_2022, fg_2023, fg_2024, fg_future);

-- 3. Tạo bảng với partition
CREATE TABLE sales_data (
    id INT,
    year INT,
    revenue DECIMAL(18,2)
) ON ps_year(year);
```

> **Lưu ý:** Phân biệt `schema` (lược đồ CSDL, ví dụ dbo) và `scheme` (lược đồ phân vùng).

---

## 34. Temporal Tables

Bảng đặc biệt lưu trữ dữ liệu lịch sử, cho phép truy vấn trạng thái dữ liệu tại bất kỳ thời điểm nào trong quá khứ.

### Tạo Temporal Table

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name NVARCHAR(100),
    salary DECIMAL(18,2),
    valid_from DATETIME2 GENERATED ALWAYS AS ROW START,
    valid_to DATETIME2 GENERATED ALWAYS AS ROW END,
    PERIOD FOR SYSTEM_TIME (valid_from, valid_to)
) WITH (SYSTEM_VERSIONING = ON);
```

### Truy vấn lịch sử

```sql
-- Dữ liệu tại thời điểm cụ thể
SELECT * FROM employees
FOR SYSTEM_TIME AS OF '2024-06-01';

-- Dữ liệu trong khoảng thời gian
SELECT * FROM employees
FOR SYSTEM_TIME BETWEEN '2024-01-01' AND '2024-12-31';
```

---

## 35. User-Defined Functions

### 35.1. Scalar Function — Trả về 1 giá trị

```sql
CREATE FUNCTION fn_calculate_tax (@price DECIMAL(18,2))
RETURNS DECIMAL(18,2)
AS
BEGIN
    RETURN @price * 0.1;
END;

-- Gọi
SELECT dbo.fn_calculate_tax(1000);  -- 100
```

> **Lưu ý:** Scalar function không được phép INSERT, UPDATE, DELETE trên bảng.

### 35.2. Inline Table-Valued Function — Trả về bảng (1 SELECT)

```sql
CREATE FUNCTION fn_get_employees_by_dept (@dept NVARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT id, name, salary
    FROM employees
    WHERE department = @dept
);

-- Gọi
SELECT * FROM fn_get_employees_by_dept(N'IT');
```

### 35.3. Multi-Statement Table-Valued Function — Trả về bảng (nhiều logic)

```sql
CREATE FUNCTION fn_get_summary (@year INT)
RETURNS @result TABLE (
    department NVARCHAR(50),
    total_salary DECIMAL(18,2)
)
AS
BEGIN
    INSERT INTO @result
    SELECT department, SUM(salary)
    FROM employees
    WHERE YEAR(hire_date) = @year
    GROUP BY department;

    RETURN;
END;
```

---

## 36. SYNONYM

Tên thay thế (alias) cho đối tượng database: view, stored procedure, function, sequence.

```sql
-- Tạo
CREATE SYNONYM short_name FOR long_database.dbo.very_long_table_name;

-- Sử dụng
SELECT * FROM short_name;

-- Xóa
DROP SYNONYM short_name;
```

---

## 37. XML

### Kiểu dữ liệu XML

SQL Server hỗ trợ kiểu dữ liệu `xml` để lưu trữ và truy vấn XML trực tiếp.

```sql
DECLARE @doc XML = '<root><item id="1">Apple</item></root>';
SELECT @doc.value('(/root/item/@id)[1]', 'INT');  -- 1
```

### FOR XML — Chuyển kết quả sang XML

```sql
SELECT * FROM employees FOR XML AUTO;
SELECT * FROM employees FOR XML PATH('employee');
```

### OPENXML — Chuyển XML sang dạng quan hệ

```sql
DECLARE @hdoc INT;
EXEC sp_xml_preparedocument @hdoc OUTPUT, @xml_data;

SELECT * FROM OPENXML(@hdoc, '/root/item')
WITH (id INT, name NVARCHAR(50));

EXEC sp_xml_removedocument @hdoc;
```

### OPENROWSET — Tải XML từ file

```sql
SELECT * FROM OPENROWSET(BULK 'C:\data\file.xml', SINGLE_BLOB) AS x;
```

---

## 38. Spatial Aggregates

Hàm tổng hợp không gian dùng cho dữ liệu địa lý (geometry, geography).

### Ứng dụng

- Tính diện tích vùng đô thị
- Xác định giao nhau giữa các khu vực
- Tạo bản đồ hiển thị vùng bao phủ
- Quản lý và quy hoạch không gian

```sql
-- Tạo POINT
DECLARE @location GEOGRAPHY = GEOGRAPHY::Point(21.0285, 105.8542, 4326);
-- (latitude, longitude, SRID)
```

---

## Tham khảo thêm

- [Microsoft T-SQL Documentation](https://learn.microsoft.com/en-us/sql/t-sql/)
- [SQL Server Tutorial](https://www.sqlservertutorial.net/)
- [W3Schools SQL](https://www.w3schools.com/sql/)
