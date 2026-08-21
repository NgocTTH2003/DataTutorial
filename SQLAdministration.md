# SQL Server Administration

Tài liệu tổng hợp kiến thức quản trị SQL Server — bao gồm mô tả ngắn gọn, câu lệnh mẫu và ví dụ thực tế.

---

## Mục lục

1. [Basics — System Databases](#section-1-basics--system-databases)
2. [Backup & Restore](#section-2-backup--restore)
   - Recovery Model
   - Backup Types
   - Full Backup
   - Differential Backup
   - Transaction Log Backup
3. [Managing Logins, Users, and Permissions](#section-3-managing-logins-users-and-permissions)
   - Create Login / User
   - Grant / Revoke Permissions
   - Alter / Drop Login & User
4. [Managing Roles](#section-4-managing-roles)
   - Create / Alter / Drop Role
5. [Database Mail](#section-5-database-mail)
6. [Blocking & Deadlock](#section-6-blocking--deadlock)
7. [Table Partitioning](#section-7-table-partitioning)
8. [Database Snapshots & Contained Databases](#section-8-database-snapshots--contained-databases)
9. [Import / Export Data](#section-9-import--export-data)
   - BCP
   - BULK INSERT
10. [Database Encryption — TDE](#section-10-database-encryption--tde)

---

## Section 1: Basics — System Databases

SQL Server có 4 system database được tạo sẵn khi cài đặt:

| Database | Mục đích |
|----------|----------|
| **master** | Lưu thông tin toàn bộ hệ thống: login, cấu hình server, danh sách database |
| **model** | Template mặc định — mỗi database mới tạo sẽ copy cấu hình từ đây |
| **msdb** | Lưu thông tin SQL Server Agent: job, schedule, backup history, alerts |
| **tempdb** | Database tạm — lưu bảng tạm, biến bảng, kết quả trung gian. Tự reset mỗi lần restart server |

```sql
-- Xem danh sách system databases
SELECT name, database_id, create_date
FROM sys.databases
WHERE database_id <= 4;
```

> **Lưu ý:** Không bao giờ xóa hoặc sửa trực tiếp system databases. Nếu master bị hỏng, SQL Server không khởi động được.

---

## Section 2: Backup & Restore

### 2.1 Recovery Model

Recovery model quyết định cách SQL Server ghi log và khả năng khôi phục dữ liệu.

| Model | Mô tả | Khi nào dùng |
|-------|--------|--------------|
| **Simple** | Tự động dọn log, không backup transaction log được | Database test, dev |
| **Full** | Giữ toàn bộ log, khôi phục đến bất kỳ thời điểm nào | Production |
| **Bulk-Logged** | Giống Full nhưng log tối thiểu cho bulk operations | Khi import data lớn |

```sql
-- Xem recovery model hiện tại
SELECT name, recovery_model_desc
FROM sys.databases;

-- Đổi recovery model
ALTER DATABASE DWH_BitCharge
SET RECOVERY FULL;
```

### 2.2 Backup Types

| Loại | Mô tả | Câu lệnh |
|------|--------|----------|
| **Full Backup** | Sao lưu toàn bộ database | `BACKUP DATABASE` |
| **Differential Backup** | Chỉ sao lưu phần thay đổi từ lần full backup gần nhất | `BACKUP DATABASE ... WITH DIFFERENTIAL` |
| **Transaction Log Backup** | Sao lưu transaction log (chỉ dùng với Full/Bulk-Logged) | `BACKUP LOG` |

### 2.3 Full Backup

Sao lưu toàn bộ database — nền tảng cho mọi chiến lược backup.

```sql
-- Full backup ra file
BACKUP DATABASE DWH_BitCharge
TO DISK = 'C:\Backup\DWH_BitCharge_Full.bak'
WITH FORMAT, INIT,
     NAME = 'Full Backup DWH_BitCharge';

-- Restore từ full backup
RESTORE DATABASE DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Full.bak'
WITH REPLACE;
```

### 2.4 Differential Backup

Chỉ backup phần data thay đổi kể từ lần full backup gần nhất — nhanh hơn, file nhỏ hơn.

```sql
-- Tạo differential backup
BACKUP DATABASE DWH_BitCharge
TO DISK = 'C:\Backup\DWH_BitCharge_Diff.bak'
WITH DIFFERENTIAL,
     NAME = 'Differential Backup DWH_BitCharge';

-- Restore: full backup trước, rồi differential
RESTORE DATABASE DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Full.bak'
WITH NORECOVERY;

RESTORE DATABASE DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Diff.bak'
WITH RECOVERY;
```

### 2.5 Transaction Log Backup

Backup log giao dịch — cho phép khôi phục đến thời điểm chính xác (point-in-time recovery).

```sql
-- Backup transaction log
BACKUP LOG DWH_BitCharge
TO DISK = 'C:\Backup\DWH_BitCharge_Log.trn'
WITH NAME = 'Log Backup DWH_BitCharge';

-- Restore: full → differential → log theo thứ tự
RESTORE DATABASE DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Full.bak'
WITH NORECOVERY;

RESTORE DATABASE DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Diff.bak'
WITH NORECOVERY;

RESTORE LOG DWH_BitCharge
FROM DISK = 'C:\Backup\DWH_BitCharge_Log.trn'
WITH RECOVERY;
```

### Chiến lược backup đề xuất cho DWH

```
Full Backup:              mỗi đêm (00:00)
Differential Backup:      mỗi 6 tiếng
Transaction Log Backup:   mỗi 30 phút (nếu dùng Full recovery model)
```

---

## Section 3: Managing Logins, Users, and Permissions

### 3.1 Phân biệt Login vs User

| Khái niệm | Phạm vi | Mục đích |
|-----------|---------|----------|
| **Login** | Server-level | Xác thực đăng nhập vào SQL Server |
| **User** | Database-level | Phân quyền truy cập trong từng database |

Một Login có thể map với nhiều User ở nhiều database khác nhau.

### 3.2 Create Login

```sql
-- Login bằng SQL Server Authentication
CREATE LOGIN analyst_login
WITH PASSWORD = 'StrongP@ss123!',
     DEFAULT_DATABASE = DWH_BitCharge;

-- Login bằng Windows Authentication
CREATE LOGIN [DOMAIN\username]
FROM WINDOWS;
```

### 3.3 Create User

```sql
-- Tạo user trong database, map với login
USE DWH_BitCharge;
CREATE USER analyst_user
FOR LOGIN analyst_login
WITH DEFAULT_SCHEMA = dbo;
```

### 3.4 Grant Permissions

```sql
-- Cho phép đọc toàn bộ bảng
GRANT SELECT ON SCHEMA::dbo TO analyst_user;

-- Cho phép đọc một bảng cụ thể
GRANT SELECT ON dbo.raw_doanh_thu_ev_power TO analyst_user;

-- Cho phép đọc + ghi
GRANT SELECT, INSERT, UPDATE ON dbo.raw_doanh_thu_ev_power TO analyst_user;

-- Cho phép thực thi stored procedure
GRANT EXECUTE ON dbo.sp_tao_summary TO analyst_user;
```

### 3.5 Revoke Permissions

```sql
-- Thu hồi quyền đã cấp
REVOKE INSERT, UPDATE ON dbo.raw_doanh_thu_ev_power FROM analyst_user;
```

### 3.6 Alter Login

```sql
-- Đổi mật khẩu
ALTER LOGIN analyst_login
WITH PASSWORD = 'NewStr0ngP@ss!';

-- Disable login
ALTER LOGIN analyst_login DISABLE;

-- Enable lại
ALTER LOGIN analyst_login ENABLE;
```

### 3.7 Alter User

```sql
-- Đổi tên user
ALTER USER analyst_user WITH NAME = report_user;

-- Đổi default schema
ALTER USER analyst_user WITH DEFAULT_SCHEMA = dbo;

-- Map sang login khác
ALTER USER analyst_user WITH LOGIN = new_login;
```

### 3.8 Drop Login & Drop User

```sql
-- Xóa user khỏi database (phải xóa user trước)
USE DWH_BitCharge;
DROP USER analyst_user;

-- Xóa login khỏi server
DROP LOGIN analyst_login;
```

> **Lưu ý:** Phải xóa user trong tất cả database trước rồi mới xóa được login.

---

## Section 4: Managing Roles

### 4.1 Roles là gì?

Role là một nhóm quyền — thay vì cấp quyền cho từng user, cấp quyền cho role rồi thêm user vào role.

SQL Server có sẵn các built-in roles:

| Role | Quyền |
|------|-------|
| **db_datareader** | Đọc tất cả bảng |
| **db_datawriter** | Ghi tất cả bảng |
| **db_owner** | Toàn quyền trên database |
| **db_ddladmin** | Tạo/sửa/xóa bảng, view, procedure |

### 4.2 Create Role

```sql
-- Tạo role tùy chỉnh
CREATE ROLE role_bao_cao;

-- Cấp quyền cho role
GRANT SELECT ON SCHEMA::dbo TO role_bao_cao;
```

### 4.3 Alter Role — Thêm/xóa thành viên

```sql
-- Thêm user vào role
ALTER ROLE role_bao_cao ADD MEMBER analyst_user;

-- Xóa user khỏi role
ALTER ROLE role_bao_cao DROP MEMBER analyst_user;

-- Đổi tên role
ALTER ROLE role_bao_cao WITH NAME = role_report;
```

### 4.4 Drop Role

```sql
-- Phải xóa hết member trước
ALTER ROLE role_bao_cao DROP MEMBER analyst_user;
DROP ROLE role_bao_cao;
```

### Ví dụ thực tế: phân quyền cho DWH

```sql
-- Role chỉ đọc báo cáo
CREATE ROLE role_report_reader;
GRANT SELECT ON dbo.summary_san_luong_thang TO role_report_reader;
GRANT SELECT ON dbo.summary_doanh_thu_tram TO role_report_reader;

-- Role ETL — đọc ghi data raw
CREATE ROLE role_etl;
GRANT SELECT, INSERT, DELETE ON dbo.raw_doanh_thu_ev_power TO role_etl;
GRANT SELECT, INSERT, DELETE ON dbo.raw_doanh_thu_rabbit_ev TO role_etl;

-- Thêm user vào role
ALTER ROLE role_report_reader ADD MEMBER analyst_user;
ALTER ROLE role_etl ADD MEMBER etl_user;
```

---

## Section 5: Database Mail

Database Mail cho phép SQL Server gửi email — thường dùng để thông báo khi job hoàn thành hoặc lỗi.

### Cấu hình Database Mail

```sql
-- Bước 1: Bật Database Mail
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'Database Mail XPs', 1;
RECONFIGURE;

-- Bước 2: Tạo Mail Profile và Account
EXEC msdb.dbo.sysmail_add_account_sp
    @account_name = 'DWH_Mail_Account',
    @email_address = 'dwh@company.com',
    @mailserver_name = 'smtp.company.com',
    @port = 587;

EXEC msdb.dbo.sysmail_add_profile_sp
    @profile_name = 'DWH_Mail_Profile';

EXEC msdb.dbo.sysmail_add_profileaccount_sp
    @profile_name = 'DWH_Mail_Profile',
    @account_name = 'DWH_Mail_Account',
    @sequence_number = 1;
```

### Gửi email

```sql
EXEC msdb.dbo.sp_send_dbmail
    @profile_name = 'DWH_Mail_Profile',
    @recipients = 'admin@company.com',
    @subject = 'ETL hoàn thành',
    @body = 'Đã load xong data tram sac vào raw.';
```

### Ứng dụng: gửi mail khi ETL xong

```sql
-- Cuối stored procedure ETL
BEGIN TRY
    -- ... logic ETL ở đây ...

    EXEC msdb.dbo.sp_send_dbmail
        @profile_name = 'DWH_Mail_Profile',
        @recipients = 'admin@company.com',
        @subject = 'ETL OK',
        @body = 'Load data thành công.';
END TRY
BEGIN CATCH
    EXEC msdb.dbo.sp_send_dbmail
        @profile_name = 'DWH_Mail_Profile',
        @recipients = 'admin@company.com',
        @subject = 'ETL LỖI',
        @body = 'Có lỗi xảy ra trong quá trình ETL.';
END CATCH
```

---

## Section 6: Blocking & Deadlock

### 6.1 Blocking

Blocking xảy ra khi một transaction giữ lock trên resource, transaction khác phải chờ.

```sql
-- Kiểm tra blocking hiện tại
SELECT
    blocking_session_id AS blocker,
    session_id AS blocked,
    wait_type,
    wait_time,
    text AS query
FROM sys.dm_exec_requests
CROSS APPLY sys.dm_exec_sql_text(sql_handle)
WHERE blocking_session_id > 0;

-- Kill session đang block (dùng khi cần thiết)
KILL 52;  -- 52 là session_id đang block
```

### Mô phỏng blocking

```sql
-- Session 1: mở transaction, không commit
BEGIN TRANSACTION;
UPDATE raw_doanh_thu_ev_power SET san_luong = 100 WHERE id = 1;
-- không COMMIT, giữ lock

-- Session 2: query cùng bảng → bị block, phải chờ
SELECT * FROM raw_doanh_thu_ev_power WHERE id = 1;

-- Giải quyết: Session 1 chạy COMMIT hoặc ROLLBACK
COMMIT;
```

### 6.2 Deadlock

Deadlock xảy ra khi 2 transaction chờ lẫn nhau — không ai nhường ai.

```
Session A: lock bảng 1, chờ bảng 2
Session B: lock bảng 2, chờ bảng 1
→ Deadlock! SQL Server tự kill một session (victim)
```

### Mô phỏng deadlock

```sql
-- Session A
BEGIN TRANSACTION;
UPDATE raw_doanh_thu_ev_power SET san_luong = 100 WHERE id = 1;
WAITFOR DELAY '00:00:05';
UPDATE raw_danh_muc_tram_sac SET ten_tram = 'Tram A' WHERE id = 1;
COMMIT;

-- Session B (chạy cùng lúc)
BEGIN TRANSACTION;
UPDATE raw_danh_muc_tram_sac SET ten_tram = 'Tram B' WHERE id = 1;
WAITFOR DELAY '00:00:05';
UPDATE raw_doanh_thu_ev_power SET san_luong = 200 WHERE id = 1;
COMMIT;

-- Kết quả: SQL Server tự detect deadlock và kill 1 session
```

### Cách tránh deadlock

```sql
-- 1. Truy cập bảng theo cùng thứ tự trong mọi transaction
-- 2. Giữ transaction ngắn nhất có thể
-- 3. Dùng NOLOCK cho query chỉ đọc (chấp nhận dirty read)
SELECT * FROM raw_doanh_thu_ev_power WITH (NOLOCK);
```

---

## Section 7: Table Partitioning

Chia một bảng lớn thành nhiều partition nhỏ theo giá trị cột — tăng performance cho query và bảo trì.

### 7.1 Tạo Partitioned Table

```sql
-- Bước 1: Tạo partition function — chia theo năm
CREATE PARTITION FUNCTION pf_theo_nam (DATE)
AS RANGE RIGHT FOR VALUES ('2024-01-01', '2025-01-01', '2026-01-01');

-- Bước 2: Tạo partition scheme — map partition vào filegroup
CREATE PARTITION SCHEME ps_theo_nam
AS PARTITION pf_theo_nam ALL TO ([PRIMARY]);

-- Bước 3: Tạo bảng với partition
CREATE TABLE raw_doanh_thu_ev_power_partitioned (
    id INT IDENTITY(1,1),
    ma_tram NVARCHAR(50),
    ngay DATE,
    san_luong DECIMAL(10,2),
    _source_file NVARCHAR(255),
    _loaded_at DATETIME2(7)
) ON ps_theo_nam(ngay);
```

### 7.2 Partition bảng đã có

```sql
-- Tạo bảng mới có partition
-- Copy data sang
-- Đổi tên bảng

-- Xem partition info
SELECT
    partition_number,
    rows
FROM sys.partitions
WHERE object_id = OBJECT_ID('raw_doanh_thu_ev_power_partitioned');
```

### Khi nào cần partition?

- Bảng có hàng triệu dòng trở lên
- Query thường lọc theo ngày/tháng/năm
- Cần xóa data cũ nhanh (drop partition thay vì DELETE)

> **Với data trạm sạc:** nếu data nhỏ thì chưa cần partition. Khi data tích lũy vài năm, hàng triệu dòng thì mới cần.

---

## Section 8: Database Snapshots & Contained Databases

### 8.1 Database Snapshot

Snapshot là bản chụp read-only của database tại một thời điểm — dùng cho báo cáo hoặc backup nhanh trước khi thay đổi lớn.

```sql
-- Tạo snapshot
CREATE DATABASE DWH_BitCharge_Snapshot
ON (
    NAME = DWH_BitCharge,
    FILENAME = 'C:\Snapshots\DWH_BitCharge_Snapshot.ss'
)
AS SNAPSHOT OF DWH_BitCharge;

-- Đọc data từ snapshot (read-only)
USE DWH_BitCharge_Snapshot;
SELECT * FROM raw_doanh_thu_ev_power;

-- Restore database từ snapshot (revert toàn bộ về thời điểm snapshot)
RESTORE DATABASE DWH_BitCharge
FROM DATABASE_SNAPSHOT = 'DWH_BitCharge_Snapshot';

-- Xóa snapshot
DROP DATABASE DWH_BitCharge_Snapshot;
```

### Khi nào dùng snapshot?

- Chụp trước khi chạy ETL lớn — nếu lỗi thì revert nhanh
- Tạo bản read-only cho team báo cáo query mà không ảnh hưởng production

### 8.2 Contained Databases

Database chứa luôn thông tin user bên trong — không phụ thuộc vào server login. Di chuyển database sang server khác dễ hơn.

```sql
-- Bật tính năng contained database trên server
EXEC sp_configure 'contained database authentication', 1;
RECONFIGURE;

-- Tạo contained database
CREATE DATABASE DWH_Contained
CONTAINMENT = PARTIAL;

-- Tạo user trực tiếp trong database (không cần login)
USE DWH_Contained;
CREATE USER contained_user
WITH PASSWORD = 'P@ssw0rd123!';

-- Cấp quyền
GRANT SELECT ON SCHEMA::dbo TO contained_user;
```

---

## Section 9: Import / Export Data

### 9.1 BCP (Bulk Copy Program)

Công cụ command line để import/export data giữa SQL Server và file.

```bash
# Export data ra CSV
bcp DWH_BitCharge.dbo.raw_doanh_thu_ev_power out "C:\Data\export.csv" -c -t"," -S localhost -U sa -P password

# Import data từ CSV
bcp DWH_BitCharge.dbo.raw_doanh_thu_ev_power in "C:\Data\import.csv" -c -t"," -S localhost -U sa -P password

# Export kết quả query
bcp "SELECT * FROM DWH_BitCharge.dbo.raw_doanh_thu_ev_power WHERE YEAR(ngay) = 2025" queryout "C:\Data\2025.csv" -c -t"," -S localhost -U sa -P password
```

Tham số phổ biến:

| Tham số | Mô tả |
|---------|--------|
| `-c` | Character mode (text) |
| `-t","` | Dấu phân cách cột là dấu phẩy |
| `-S` | Server name |
| `-U` | Username |
| `-P` | Password |
| `-T` | Dùng Windows Authentication |

### 9.2 BULK INSERT

Cú pháp:

```
BULK INSERT table_name
FROM path_to_file
WITH options;
```

Câu lệnh T-SQL để import file vào bảng — chạy trong SSMS.

```sql
-- Import CSV vào bảng
BULK INSERT raw_doanh_thu_ev_power
FROM 'C:\Data\doanh_thu_ev_power.csv'
WITH (
    FIRSTROW = 2,              -- bỏ qua header
    FIELDTERMINATOR = ',',     -- dấu phân cách cột
    ROWTERMINATOR = '\n',      -- dấu xuống dòng
    CODEPAGE = '65001',        -- UTF-8 (hỗ trợ tiếng Việt)
    TABLOCK                    -- lock toàn bảng, nhanh hơn
);
```

### Import nhiều file cùng lúc

```sql
-- Dùng xp_cmdshell + vòng lặp (cần bật xp_cmdshell)
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Hoặc đơn giản: chạy BULK INSERT cho từng file
BULK INSERT raw_doanh_thu_ev_power
FROM 'C:\Data\thang1.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', ROWTERMINATOR = '\n');

BULK INSERT raw_doanh_thu_ev_power
FROM 'C:\Data\thang2.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', ROWTERMINATOR = '\n');
```

### So sánh BCP vs BULK INSERT

| | BCP | BULK INSERT |
|--|-----|-------------|
| Chạy ở đâu | Command line | SSMS / T-SQL |
| Export được không | Có | Không |
| Dùng trong stored procedure | Không trực tiếp | Có |
| Phù hợp | Automation, script | Ad-hoc import |

---

## Section 10: Database Encryption — TDE

Transparent Data Encryption (TDE) mã hóa file database trên ổ đĩa — bảo vệ data khi ai đó copy file .mdf/.ldf.

```sql
-- Bước 1: Tạo master key trong database master
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MasterK3y!Str0ng';

-- Bước 2: Tạo certificate
CREATE CERTIFICATE TDE_Cert
WITH SUBJECT = 'TDE Certificate for DWH_BitCharge';

-- Bước 3: Tạo database encryption key
USE DWH_BitCharge;
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDE_Cert;

-- Bước 4: Bật encryption
ALTER DATABASE DWH_BitCharge SET ENCRYPTION ON;

-- Kiểm tra trạng thái
SELECT db.name, db.is_encrypted, ek.encryption_state
FROM sys.databases db
LEFT JOIN sys.dm_database_encryption_keys ek
    ON db.database_id = ek.database_id;
```

### Backup certificate (RẤT QUAN TRỌNG)

```sql
-- Nếu mất certificate = mất data
BACKUP CERTIFICATE TDE_Cert
TO FILE = 'C:\Backup\TDE_Cert.cer'
WITH PRIVATE KEY (
    FILE = 'C:\Backup\TDE_Cert_Key.pvk',
    ENCRYPTION BY PASSWORD = 'BackupP@ss123!'
);
```

> **Lưu ý:** Backup certificate và giữ ở nơi an toàn. Không có certificate thì không restore được database đã mã hóa.

### Khi nào cần TDE?

- Database chứa dữ liệu nhạy cảm (thông tin khách hàng, tài chính)
- Yêu cầu compliance (GDPR, PCI-DSS)
- Server có nguy cơ bị truy cập vật lý

---

## Tổng kết: Áp dụng cho DWH Trạm Sạc

| Section | Áp dụng cho DWH BitCharge |
|---------|--------------------------|
| System Databases | Hiểu cấu trúc server |
| Backup & Restore | Backup hàng đêm cho DWH |
| Logins & Users | Tạo account cho ETL, analyst, report |
| Roles | role_etl (đọc/ghi raw), role_report (chỉ đọc summary) |
| Database Mail | Gửi thông báo khi ETL xong hoặc lỗi |
| Blocking & Deadlock | Tránh xung đột khi ETL và report chạy cùng lúc |
| Table Partitioning | Partition theo tháng/năm khi data lớn |
| Database Snapshot | Chụp trước khi chạy ETL lớn |
| Import/Export | BULK INSERT để load CSV vào raw |
| TDE | Mã hóa nếu có data khách hàng nhạy cảm |
