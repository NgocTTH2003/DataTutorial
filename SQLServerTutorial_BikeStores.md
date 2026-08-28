# Lộ trình học SQL Server từ cơ bản đến nâng cao

Link: https://www.sqlservertutorial.net/

## SQL Server Basics

### Section 1: Querying data
SELECT

### Section 2: Sorting data
ORDER BY

### Section 3: Limiting rows
OFFSET FETCH, SELECT TOP

#### 3.1.Mệnh đề OFFSET FETCH
```
ORDER BY column_list [ASC |DESC]
OFFSET offset_row_count {ROW | ROWS}
FETCH {FIRST | NEXT} fetch_row_count {ROW | ROWS} ONLY
```

- Mệnh đề này OFFSET chỉ định số lượng hàng cần bỏ qua trước khi bắt đầu trả về các hàng từ truy vấn. Giá trị này offset_row_countcó thể là một hằng số, biến số hoặc tham số lớn hơn hoặc bằng không.
- Mệnh đề này FETCH chỉ định số lượng hàng cần trả về sau khi OFFSET mệnh đề đã được xử lý. Giá trị này offset_row_count ó thể là một hằng số, biến số hoặc một số vô hướng lớn hơn hoặc bằng một.
- Mệnh đề này OFFSET là bắt buộc, trong khi FETCH mệnh đề kia là tùy chọn. Ngoài ra, FIRST và NEXT là từ đồng nghĩa và có thể được sử dụng thay thế cho nhau. Tương tự, bạn có thể sử dụng ROW và ROWS thay thế cho nhau.

### Section 4: Filtering data
DISTINCT, WHERE, AND, OR, IN, BETWEEN, LIKE, Column & table aliases

### Section 5: Joining tables
Joins, INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN, CROSS JOIN, Self join
