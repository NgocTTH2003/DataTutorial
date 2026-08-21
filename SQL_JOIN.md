# SQL JOIN

> JOIN kết hợp dữ liệu từ 2+ bảng dựa trên cột liên quan.

---

## 1. Tổng quan

| JOIN | Trả về | NULL? |
|---|---|---|
| `INNER JOIN` | Chỉ hàng khớp cả 2 bảng | Không |
| `LEFT JOIN` | Tất cả bảng trái + khớp bảng phải | Bảng phải = NULL nếu không khớp |
| `RIGHT JOIN` | Tất cả bảng phải + khớp bảng trái | Bảng trái = NULL nếu không khớp |
| `FULL JOIN` | Tất cả cả 2 bảng | NULL ở bên không khớp |
| `CROSS JOIN` | Tích Descartes (A × B) | Không, không cần ON |
| `SELF JOIN` | Bảng join chính nó | Tùy loại JOIN |

---

## 2. Dữ liệu mẫu

**employees:**

| id | name | department_id |
|---|---|---|
| 1 | An | 10 |
| 2 | Bình | 20 |
| 3 | Cường | 10 |
| 4 | Dũng | NULL |
| 5 | Em | 30 |

**departments:**

| id | dept_name |
|---|---|
| 10 | IT |
| 20 | HR |
| 40 | Finance |

> Dũng không có dept. Em thuộc dept 30 (không tồn tại). Finance không có nhân viên.

---

## 3. INNER JOIN — Chỉ hàng khớp

```sql
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

→ **3 hàng:** An-IT, Bình-HR, Cường-IT. Dũng, Em, Finance bị loại.

---

## 4. LEFT JOIN — Giữ tất cả bảng trái

```sql
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

→ **5 hàng:** 3 khớp + Dũng (NULL) + Em (NULL). Finance không xuất hiện.

**Anti Join — chỉ lấy hàng KHÔNG khớp:**

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

→ Dũng, Em (dùng để tìm dữ liệu "mồ côi")

---

## 5. RIGHT JOIN — Giữ tất cả bảng phải

```sql
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

→ **4 hàng:** 3 khớp + Finance (name = NULL). Dũng, Em không xuất hiện.

> Thực tế ít dùng — đổi thứ tự bảng rồi dùng LEFT JOIN cho dễ đọc.

---

## 6. FULL JOIN — Giữ tất cả cả 2 bảng

```sql
SELECT e.name, d.dept_name
FROM employees e
FULL JOIN departments d ON e.department_id = d.id;
```

→ **6 hàng:** 3 khớp + Dũng (NULL) + Em (NULL) + Finance (NULL). Không mất gì.

---

## 7. CROSS JOIN — Tích Descartes

```sql
SELECT e.name, d.dept_name
FROM employees e
CROSS JOIN departments d;
```

→ **15 hàng** (5 × 3). Không cần ON. Cẩn thận với bảng lớn!

---

## 8. SELF JOIN — Bảng join chính nó

```sql
-- Tìm quản lý
SELECT e.name AS nhan_vien, m.name AS quan_ly
FROM staff e
LEFT JOIN staff m ON e.manager_id = m.id;

-- Tìm cặp nhân viên cùng phòng (dùng a.id < b.id tránh trùng cặp)
SELECT a.name, b.name
FROM employees a
INNER JOIN employees b ON a.department_id = b.department_id
WHERE a.id < b.id;
```

---

## 9. ON vs WHERE — Khác nhau quan trọng

```sql
-- Điều kiện trong ON (lọc TRƯỚC khi join)
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id AND d.dept_name = 'IT';
-- → Tất cả employees, chỉ dept IT khớp, còn lại NULL

-- Điều kiện trong WHERE (lọc SAU khi join)
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.dept_name = 'IT';
-- → Chỉ employees thuộc IT (giống INNER JOIN!)
```

> **Lưu ý:** Với INNER JOIN thì ON và WHERE giống nhau. Với LEFT/RIGHT/FULL JOIN thì **khác nhau hoàn toàn**. Hay ra bài test!

---

## 10. Lỗi thường gặp

| Lỗi | Hậu quả | Cách sửa |
|---|---|---|
| Thiếu ON | Thành CROSS JOIN | Luôn có ON (trừ CROSS JOIN) |
| JOIN sai cột | Kết quả sai | Kiểm tra đúng PK-FK |
| WHERE trên LEFT JOIN | Mất NULL, thành INNER JOIN | Chuyển điều kiện vào ON |
| Không dùng alias | Lỗi ambiguous column | Luôn đặt alias cho bảng |
| SELF JOIN không tránh trùng | Cặp xuất hiện 2 lần | Dùng `a.id < b.id` |

---

## 11. Tóm tắt cách chọn JOIN

```
Cần dữ liệu khớp cả 2 bảng?
├── Có → INNER JOIN
└── Không → Giữ bảng nào?
    ├── Bảng trái → LEFT JOIN
    ├── Bảng phải → RIGHT JOIN
    ├── Cả hai → FULL JOIN
    └── Mọi tổ hợp → CROSS JOIN
```
