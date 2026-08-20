# Hướng dẫn chuyển dữ liệu từ DB1 (Server A) sang DB2 (Server B)

## Hai trường hợp chính

| Trường hợp | Ví dụ | Các cách khả thi |
|---|---|---|
| **TH1: Cùng hệ quản trị** | SQL Server ↔ SQL Server | Linked Server, Replication, CDC, Python |
| **TH2: Khác hệ quản trị** | SQL Server → MySQL / PostgreSQL / MongoDB | Linked Server + ODBC, Python, ETL Tool |

---

# TRƯỜNG HỢP 1: CÙNG HỆ QUẢN TRỊ (SQL Server ↔ SQL Server)

## 1.1. Linked Server + SQL Agent

### Tổng quan

Linked Server cho phép 2 SQL Server kết nối với nhau. Có 2 cách chuyển data từ A sang B:

- **Push (đẩy)**: tạo Linked Server trên Server A, đẩy data sang B
- **Pull (kéo)**: tạo Linked Server trên Server B, kéo data từ A về

Cả 2 cách đều được. Linked Server tạo trên server nào thì chạy query trên server đó.

- **Kết nối**: Linked Server
- **ETL xử lý trên**: SQL Server (stored procedure)
- **Tự động hóa**: SQL Agent Job
- **Phù hợp**: data không quá lớn, logic transform đơn giản, cùng hệ SQL Server

---

### Cách A: PUSH — Đứng trên Server A, đẩy sang B

#### Bước 1 — Tạo Linked Server trên Server A (trỏ sang B)

```sql
-- Chạy trên Server A
EXEC sp_addlinkedserver
    @server = 'SERVER_B',
    @srvproduct = '',
    @provider = 'SQLNCLI',
    @datasrc = '192.168.1.200';  -- IP Server B

EXEC sp_addlinkedsrvlogin
    @rmtsrvname = 'SERVER_B',
    @useself = 'FALSE',
    @rmtuser = 'sa',
    @rmtpassword = 'password_server_b';

-- Kiểm tra
EXEC sp_testlinkedserver 'SERVER_B';
```

#### Bước 2 — Stored Procedure đẩy data sang B

```sql
-- Chạy trên Server A: đẩy data từ A → B
-- Full load (lần đầu)
CREATE PROCEDURE sp_push_full_load
AS
BEGIN
    -- Xóa data cũ trên Server B
    EXEC('TRUNCATE TABLE [DB_Dich].[dbo].[raw_doanh_thu]') AT SERVER_B;

    -- Đẩy toàn bộ data từ A sang B
    INSERT INTO [SERVER_B].[DB_Dich].[dbo].[raw_doanh_thu]
    SELECT *
    FROM [DB_Nguon].[dbo].[doanh_thu];

    PRINT N'Push full load: ' + CAST(@@ROWCOUNT AS NVARCHAR) + N' dòng';
END
GO
```

```sql
-- Incremental load (hàng ngày)
CREATE PROCEDURE sp_push_incremental
AS
BEGIN
    DECLARE @max_date DATE;

    -- Lấy ngày mới nhất đã có trên Server B
    SELECT @max_date = MAX(ngay_thanh_toan)
    FROM [SERVER_B].[DB_Dich].[dbo].[raw_doanh_thu];

    -- Chỉ đẩy data mới hơn
    INSERT INTO [SERVER_B].[DB_Dich].[dbo].[raw_doanh_thu]
    SELECT *
    FROM [DB_Nguon].[dbo].[doanh_thu]
    WHERE ngay_thanh_toan > @max_date;

    PRINT N'Push incremental: ' + CAST(@@ROWCOUNT AS NVARCHAR) + N' dòng mới';
END
GO
```

#### Sơ đồ

```
Server A (chạy query ở đây)              Server B
┌────────────────────────┐            ┌──────────────────┐
│  DB_Nguon              │            │  DB_Dich         │
│  ┌──────────────┐      │  Linked    │  ┌─────────────┐ │
│  │  doanh_thu   │──────│──Server──▶ │  │raw_doanh_thu│ │
│  └──────────────┘      │   PUSH     │  └─────────────┘ │
│                        │            │                  │
│  sp_push_incremental   │            │                  │
│  SQL Agent (6AM daily) │            │                  │
└────────────────────────┘            └──────────────────┘
```

---

### Cách B: PULL — Đứng trên Server B, kéo từ A về

#### Bước 1 — Tạo Linked Server trên Server B (trỏ sang A)

```sql
-- Chạy trên Server B
EXEC sp_addlinkedserver
    @server = 'SERVER_A',
    @srvproduct = '',
    @provider = 'SQLNCLI',
    @datasrc = '192.168.1.100';  -- IP Server A

EXEC sp_addlinkedsrvlogin
    @rmtsrvname = 'SERVER_A',
    @useself = 'FALSE',
    @rmtuser = 'sa',
    @rmtpassword = 'password_server_a';

-- Kiểm tra
EXEC sp_testlinkedserver 'SERVER_A';
```

#### Bước 2 — Stored Procedure kéo data từ A về

```sql
-- Chạy trên Server B: kéo data từ A về B
-- Full load (lần đầu)
CREATE PROCEDURE sp_pull_full_load
AS
BEGIN
    TRUNCATE TABLE [DB_Dich].[dbo].[raw_doanh_thu];

    INSERT INTO [DB_Dich].[dbo].[raw_doanh_thu]
    SELECT *
    FROM [SERVER_A].[DB_Nguon].[dbo].[doanh_thu];

    PRINT N'Pull full load: ' + CAST(@@ROWCOUNT AS NVARCHAR) + N' dòng';
END
GO
```

```sql
-- Incremental load (hàng ngày)
CREATE PROCEDURE sp_pull_incremental
AS
BEGIN
    DECLARE @max_date DATE;

    SELECT @max_date = MAX(ngay_thanh_toan)
    FROM [DB_Dich].[dbo].[raw_doanh_thu];

    INSERT INTO [DB_Dich].[dbo].[raw_doanh_thu]
    SELECT *
    FROM [SERVER_A].[DB_Nguon].[dbo].[doanh_thu]
    WHERE ngay_thanh_toan > @max_date;

    PRINT N'Pull incremental: ' + CAST(@@ROWCOUNT AS NVARCHAR) + N' dòng mới';
END
GO
```

#### Sơ đồ

```
Server A                              Server B (chạy query ở đây)
┌──────────────────┐               ┌────────────────────────┐
│  DB_Nguon        │               │  DB_Dich               │
│  ┌─────────────┐ │    Linked     │  ┌──────────────┐      │
│  │  doanh_thu  │ │◀──Server──────│  │ raw_doanh_thu│      │
│  └─────────────┘ │     PULL      │  └──────────────┘      │
│                  │               │                        │
│                  │               │  sp_pull_incremental   │
│                  │               │  SQL Agent (6AM daily) │
└──────────────────┘               └────────────────────────┘
```

---

### Push vs Pull — Nên chọn cách nào?

| Tiêu chí | Push (từ A) | Pull (từ B) |
|---|---|---|
| Linked Server tạo trên | Server A | Server B |
| SQL Agent chạy trên | Server A | Server B |
| Kiểm soát lịch chạy | Team quản lý A | Team quản lý B |
| Phù hợp khi | A chủ động gửi data | B chủ động lấy data |
| Thường dùng hơn | Ít | **Nhiều hơn** (B cần data thì B tự lấy) |

Trong thực tế, **Pull phổ biến hơn** vì server cần data sẽ tự chủ động lấy, không phụ thuộc vào server nguồn.

---

### Bước 3 — Tạo SQL Agent Job (áp dụng cho cả Push và Pull)

```sql
-- Tạo job (chạy trên server nào tạo Linked Server)
EXEC msdb.dbo.sp_add_job
    @job_name = N'Job_Sync_Doanh_Thu',
    @enabled = 1,
    @description = N'Tự động đồng bộ doanh thu mỗi ngày';

-- Thêm step: chạy stored procedure
-- Đổi tên procedure tùy Push hoặc Pull
EXEC msdb.dbo.sp_add_jobstep
    @job_name = N'Job_Sync_Doanh_Thu',
    @step_name = N'Chay_Incremental_Load',
    @subsystem = N'TSQL',
    @command = N'EXEC sp_pull_incremental',  -- hoặc sp_push_incremental
    @database_name = N'DB_Dich';

-- Hẹn giờ: chạy mỗi ngày lúc 6:00 sáng
EXEC msdb.dbo.sp_add_schedule
    @schedule_name = N'Daily_6AM',
    @freq_type = 4,
    @freq_interval = 1,
    @active_start_time = 060000;

-- Gán schedule vào job
EXEC msdb.dbo.sp_attach_schedule
    @job_name = N'Job_Sync_Doanh_Thu',
    @schedule_name = N'Daily_6AM';

-- Gán job cho server
EXEC msdb.dbo.sp_add_jobserver
    @job_name = N'Job_Sync_Doanh_Thu',
    @server_name = N'(local)';
```

### Ưu / nhược điểm

- Ưu: đơn giản, không cần tool bên ngoài, xử lý hoàn toàn trong SQL
- Nhược: chỉ hoạt động tốt giữa các SQL Server, data quá lớn có thể chậm qua network

---

## 1.2. Replication (Sao chép dữ liệu)

### Tổng quan

SQL Server tự động đồng bộ data từ Server A sang Server B. Có 3 loại:

- **Snapshot Replication**: copy toàn bộ data theo lịch (ví dụ mỗi đêm)
- **Transactional Replication**: đồng bộ realtime, mỗi thay đổi trên A tự động chạy sang B
- **Merge Replication**: cả 2 bên đều có thể ghi, tự merge lại

- **Kết nối**: SQL Server tự quản lý
- **ETL xử lý trên**: không cần ETL, data được sao chép nguyên trạng
- **Tự động hóa**: hoàn toàn tự động sau khi cấu hình
- **Phù hợp**: cần data đồng bộ liên tục, không cần transform

### Cấu hình Transactional Replication (qua SSMS)

Bước 1: Trên Server A (Publisher — nơi có data gốc)

```
SSMS → Replication → Local Publications → click phải
→ New Publication
→ Chọn database nguồn (DB_Nguon)
→ Chọn loại: Transactional
→ Chọn bảng cần replicate (ví dụ: doanh_thu)
→ Finish
```

Bước 2: Trên Server B (Subscriber — nơi nhận data)

```
SSMS → Replication → Local Subscriptions → click phải
→ New Subscription
→ Chọn Publisher: Server A
→ Chọn Publication vừa tạo
→ Chọn database đích trên Server B (DB_Dich)
→ Chọn lịch đồng bộ (liên tục hoặc theo giờ)
→ Finish
```

### Cấu hình bằng T-SQL

```sql
-- Trên Server A (Publisher): bật distribution
EXEC sp_adddistributor @distributor = N'SERVER_A';
EXEC sp_adddistributiondb @database = N'distribution';

-- Tạo publication
EXEC sp_addpublication
    @publication = N'Pub_Doanh_Thu',
    @status = N'active';

-- Thêm bảng vào publication
EXEC sp_addarticle
    @publication = N'Pub_Doanh_Thu',
    @article = N'doanh_thu',
    @source_table = N'doanh_thu',
    @source_object = N'doanh_thu';

-- Trên Server B (Subscriber): đăng ký nhận data
EXEC sp_addsubscription
    @publication = N'Pub_Doanh_Thu',
    @subscriber = N'SERVER_B',
    @destination_db = N'DB_Dich';
```

### Sơ đồ luồng

```
Server A (Publisher)                Server B (Subscriber)
┌──────────────┐   Tự động       ┌──────────────┐
│  doanh_thu   │──đồng bộ────▶  │  doanh_thu   │
│  (data gốc)  │   realtime     │  (bản sao)   │
└──────────────┘                 └──────────────┘
        │                                │
   Mỗi INSERT/                    Data tự xuất
   UPDATE/DELETE                   hiện bên này
   tự chạy sang B
```

### Ưu / nhược điểm

- Ưu: hoàn toàn tự động, gần realtime, không cần viết code
- Nhược: chỉ copy nguyên data (không transform được), cấu hình ban đầu phức tạp, tốn tài nguyên server

---

## 1.3. CDC — Change Data Capture

### Tổng quan

CDC theo dõi mọi thay đổi (INSERT, UPDATE, DELETE) trên bảng nguồn ở Server A và ghi lại vào bảng log. Từ bảng log đó, chỉ lấy phần data thay đổi để đẩy sang Server B.

- **Kết nối**: Linked Server (hoặc Python)
- **ETL xử lý trên**: SQL Server (stored procedure đọc từ bảng CDC)
- **Tự động hóa**: SQL Agent Job
- **Phù hợp**: data rất lớn, chỉ muốn đồng bộ phần thay đổi, cần biết ai thay đổi gì

### Bước 1 — Bật CDC trên Server A (nơi có data gốc)

```sql
-- Bật CDC cho database
USE DB_Nguon;
EXEC sys.sp_cdc_enable_db;

-- Bật CDC cho bảng cần theo dõi
EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo',
    @source_name = N'doanh_thu',
    @role_name = NULL;
```

### Bước 2 — Xem data thay đổi trên Server A

```sql
DECLARE @from_lsn BINARY(10), @to_lsn BINARY(10);

SET @from_lsn = sys.fn_cdc_get_min_lsn('dbo_doanh_thu');
SET @to_lsn = sys.fn_cdc_get_max_lsn();

SELECT *
FROM cdc.fn_cdc_get_all_changes_dbo_doanh_thu(
    @from_lsn, @to_lsn, N'all'
);
```

Kết quả trả về gồm cột `__$operation`:
- 1 = DELETE
- 2 = INSERT
- 3 = UPDATE (giá trị cũ)
- 4 = UPDATE (giá trị mới)

### Bước 3 — Tạo Procedure trên Server B đồng bộ data thay đổi

```sql
-- Chạy trên Server B (đã tạo Linked Server trỏ sang A)
CREATE PROCEDURE sp_cdc_sync_doanh_thu
AS
BEGIN
    DECLARE @from_lsn BINARY(10), @to_lsn BINARY(10);

    -- Lấy phạm vi thay đổi từ Server A
    SELECT @from_lsn = sys.fn_cdc_get_min_lsn('dbo_doanh_thu');
    SELECT @to_lsn = sys.fn_cdc_get_max_lsn();

    -- Kéo các dòng INSERT hoặc UPDATE từ Server A về
    INSERT INTO [DB_Dich].[dbo].[raw_doanh_thu]
    SELECT
        ma_don_hang, so_dien_thoai, tong_phi_thanh_toan,
        ngay_thanh_toan
    FROM OPENQUERY(SERVER_A,
        'SELECT * FROM cdc.fn_cdc_get_net_changes_dbo_doanh_thu(
            sys.fn_cdc_get_min_lsn(''dbo_doanh_thu''),
            sys.fn_cdc_get_max_lsn(),
            ''all''
        ) WHERE __$operation IN (2, 4)'
    );

    PRINT N'CDC sync: ' + CAST(@@ROWCOUNT AS NVARCHAR) + N' dòng';
END
GO
```

### Sơ đồ luồng

```
Server A (data gốc)
┌──────────────────────────────────┐
│  doanh_thu (bảng gốc)           │
│       │                          │
│       ▼                          │
│  CDC log table                   │
│  (tự ghi INSERT/UPDATE/DELETE)   │
└──────────────┬───────────────────┘
               │ Linked Server (Pull)
               ▼
Server B (nhận data)
┌──────────────────────────────────┐
│  sp_cdc_sync_doanh_thu          │
│       │                          │
│       ▼                          │
│  raw_doanh_thu (chỉ data mới)   │
│                                  │
│  SQL Agent (mỗi 30 phút)        │
└──────────────────────────────────┘
```

### Ưu / nhược điểm

- Ưu: chỉ đồng bộ data thay đổi (nhanh, tiết kiệm), biết chính xác dòng nào thay đổi
- Nhược: tốn thêm storage cho CDC log, cấu hình phức tạp, cần SQL Server Enterprise hoặc Developer edition

---

# TRƯỜNG HỢP 2: KHÁC HỆ QUẢN TRỊ (SQL Server → MySQL / PostgreSQL / MongoDB...)

## 2.1. Linked Server + ODBC Driver

### Tổng quan

SQL Server vẫn dùng được Linked Server để connect sang MySQL/PostgreSQL, nhưng cần cài thêm ODBC Driver làm cầu nối.

- **Kết nối**: Linked Server + ODBC Driver
- **ETL xử lý trên**: SQL Server (stored procedure)
- **Tự động hóa**: SQL Agent Job
- **Phù hợp**: query đơn giản, data nhỏ-vừa

### Bước 1 — Cài ODBC Driver

Ví dụ: SQL Server (A) → MySQL (B)

```
1. Download MySQL ODBC Driver: https://dev.mysql.com/downloads/connector/odbc/
2. Cài đặt trên máy chạy SQL Server (Server A)
3. Mở ODBC Data Sources (64-bit)
4. Add → chọn MySQL ODBC Driver
5. Điền: Server B IP, Port (3306), Database, User, Password
6. Đặt tên DSN: MYSQL_DSN
7. Test Connection → OK
```

Ví dụ: SQL Server (A) → PostgreSQL (B)

```
1. Download PostgreSQL ODBC Driver: https://www.postgresql.org/ftp/odbc/
2. Cài đặt trên máy chạy SQL Server (Server A)
3. Mở ODBC Data Sources (64-bit)
4. Add → chọn PostgreSQL Unicode
5. Điền: Server B IP, Port (5432), Database, User, Password
6. Đặt tên DSN: POSTGRES_DSN
7. Test Connection → OK
```

### Bước 2 — Tạo Linked Server qua ODBC

```sql
-- SQL Server (A) → MySQL (B)
EXEC sp_addlinkedserver
    @server = 'MYSQL_SERVER_B',
    @srvproduct = 'MySQL',
    @provider = 'MSDASQL',
    @datasrc = 'MYSQL_DSN';

EXEC sp_addlinkedsrvlogin
    @rmtsrvname = 'MYSQL_SERVER_B',
    @useself = 'FALSE',
    @rmtuser = 'root',
    @rmtpassword = 'mysql_password';

EXEC sp_testlinkedserver 'MYSQL_SERVER_B';
```

```sql
-- SQL Server (A) → PostgreSQL (B)
EXEC sp_addlinkedserver
    @server = 'PG_SERVER_B',
    @srvproduct = 'PostgreSQL',
    @provider = 'MSDASQL',
    @datasrc = 'POSTGRES_DSN';

EXEC sp_addlinkedsrvlogin
    @rmtsrvname = 'PG_SERVER_B',
    @useself = 'FALSE',
    @rmtuser = 'postgres',
    @rmtpassword = 'pg_password';
```

### Bước 3 — Đẩy data từ SQL Server (A) sang MySQL/PostgreSQL (B)

```sql
-- Đọc data từ MySQL (B) về SQL Server (A)
SELECT * FROM OPENQUERY(MYSQL_SERVER_B, 'SELECT * FROM doanh_thu');

-- Đẩy data từ SQL Server (A) sang MySQL (B)
INSERT INTO OPENQUERY(MYSQL_SERVER_B,
    'SELECT ma_don, ten_sp, so_luong, ngay FROM doanh_thu')
SELECT ma_don, ten_sp, so_luong, ngay
FROM [DB_Nguon].[dbo].[doanh_thu]
WHERE ngay > '2024-01-01';
```

Lưu ý: phải dùng `OPENQUERY()` vì cross-platform không hỗ trợ cú pháp 4 phần `[Server].[DB].[Schema].[Table]`.

### Sơ đồ luồng

```
SQL Server (A)                     MySQL / PostgreSQL (B)
┌──────────────────────┐           ┌──────────────┐
│  DB_Nguon            │   ODBC +  │  database_b  │
│  ┌─────────────┐     │  Linked   │  ┌─────────┐ │
│  │  doanh_thu  │─────│──Server──▶│  │ doanh_thu│ │
│  └─────────────┘     │   PUSH    │  └─────────┘ │
│                      │           │              │
│  SQL Agent (daily)   │           │              │
└──────────────────────┘           └──────────────┘
```

### Ưu / nhược điểm

- Ưu: vẫn xử lý trong SQL Server, không cần code thêm
- Nhược: cần cài ODBC driver, OPENQUERY có giới hạn syntax, performance kém với data lớn

---

## 2.2. Python Pipeline (khuyến nghị cho cross-platform)

### Tổng quan

Python là lựa chọn tốt nhất khi 2 DB khác hệ quản trị, vì mỗi DB có driver riêng và Python hỗ trợ hết.

- **Kết nối**: Python (thư viện driver cho từng DB)
- **ETL xử lý trên**: Python (pandas)
- **Tự động hóa**: Windows Task Scheduler hoặc cron job
- **Phù hợp**: cross-platform, cần transform phức tạp, kết hợp nhiều nguồn

### Thư viện driver theo từng DB

| DB | Thư viện Python | Cài đặt |
|---|---|---|
| SQL Server | pyodbc | `pip install pyodbc` |
| MySQL | pymysql | `pip install pymysql` |
| PostgreSQL | psycopg2 | `pip install psycopg2-binary` |
| MongoDB | pymongo | `pip install pymongo` |
| Oracle | cx_Oracle | `pip install cx_Oracle` |
| SQLite | sqlite3 | (có sẵn trong Python) |

### Ví dụ 1: SQL Server (A) → MySQL (B)

```python
import pandas as pd
from sqlalchemy import create_engine

# Kết nối SQL Server - Server A (nguồn)
engine_a = create_engine(
    'mssql+pyodbc://sa:password@192.168.1.100/DB_Nguon'
    '?driver=ODBC+Driver+17+for+SQL+Server'
)

# Kết nối MySQL - Server B (đích)
engine_b = create_engine(
    'mysql+pymysql://root:password@192.168.1.200:3306/DB_Dich'
)

# Đọc từ A
df = pd.read_sql('SELECT * FROM doanh_thu WHERE ngay > "2024-01-01"', engine_a)

# Transform nếu cần
df['tong_tien'] = df['tong_tien'].astype(float)
df['ngay'] = pd.to_datetime(df['ngay'])

# Ghi vào B
df.to_sql('raw_doanh_thu', engine_b, if_exists='append', index=False)
print(f'Đã chuyển {len(df)} dòng từ SQL Server (A) → MySQL (B)')
```

### Ví dụ 2: SQL Server (A) → PostgreSQL (B)

```python
import pandas as pd
from sqlalchemy import create_engine

# Kết nối SQL Server - Server A (nguồn)
engine_a = create_engine(
    'mssql+pyodbc://sa:password@192.168.1.100/DB_Nguon'
    '?driver=ODBC+Driver+17+for+SQL+Server'
)

# Kết nối PostgreSQL - Server B (đích)
engine_b = create_engine(
    'postgresql+psycopg2://postgres:password@192.168.1.200:5432/DB_Dich'
)

# Đọc từ A
df = pd.read_sql('SELECT * FROM doanh_thu', engine_a)

# Ghi vào B
df.to_sql('raw_doanh_thu', engine_b, if_exists='append', index=False)
print(f'Đã chuyển {len(df)} dòng từ SQL Server (A) → PostgreSQL (B)')
```

### Ví dụ 3: SQL Server (A) → MongoDB (B)

```python
import pandas as pd
from sqlalchemy import create_engine
from pymongo import MongoClient

# Kết nối SQL Server - Server A (nguồn)
engine_a = create_engine(
    'mssql+pyodbc://sa:password@192.168.1.100/DB_Nguon'
    '?driver=ODBC+Driver+17+for+SQL+Server'
)

# Kết nối MongoDB - Server B (đích)
client = MongoClient('mongodb://192.168.1.200:27017/')
db_b = client['DB_Dich']
collection = db_b['raw_doanh_thu']

# Đọc từ A
df = pd.read_sql('SELECT * FROM doanh_thu', engine_a)

# Ghi vào MongoDB (B)
records = df.to_dict('records')
collection.insert_many(records)
print(f'Đã chuyển {len(records)} dòng từ SQL Server (A) → MongoDB (B)')
```

### Ví dụ 4: SQL Server (A) → SQL Server (B) — Incremental Load

```python
import pandas as pd
from sqlalchemy import create_engine, text

# Kết nối Server A (nguồn)
engine_a = create_engine(
    'mssql+pyodbc://sa:password@192.168.1.100/DB_Nguon'
    '?driver=ODBC+Driver+17+for+SQL+Server'
)

# Kết nối Server B (đích)
engine_b = create_engine(
    'mssql+pyodbc://sa:password@192.168.1.200/DB_Dich'
    '?driver=ODBC+Driver+17+for+SQL+Server'
)

# Lấy ngày mới nhất trên Server B
with engine_b.connect() as conn:
    result = conn.execute(text('SELECT MAX(ngay_thanh_toan) FROM raw_doanh_thu'))
    max_date = result.scalar()

# Chỉ đọc data mới từ Server A
df = pd.read_sql(f'''
    SELECT * FROM doanh_thu
    WHERE ngay_thanh_toan > '{max_date}'
''', engine_a)

if len(df) > 0:
    df.to_sql('raw_doanh_thu', engine_b, if_exists='append', index=False)
    print(f'Đã đẩy {len(df)} dòng mới từ A → B')
else:
    print('Không có data mới')
```

### Tự động hóa bằng Task Scheduler

```
1. Mở Task Scheduler
2. Create Basic Task → tên: "ETL_A_to_B"
3. Trigger: Daily → 06:00 AM
4. Action: Start a Program
   - Program: C:\Python311\python.exe
   - Arguments: C:\scripts\etl_a_to_b.py
5. Finish
```

Hoặc bằng command line:

```cmd
schtasks /create /tn "ETL_A_to_B" /tr "python C:\scripts\etl_a_to_b.py" /sc daily /st 06:00
```

### Sơ đồ luồng

```
Server A (nguồn)                   Python               
┌──────────────┐   pyodbc/     ┌──────────────┐  pymysql/Server B (đích)  ┌──────────────┐
│  doanh_thu   │──sqlalchemy──▶│  pandas      │──pyodbc────────────────▶ │ raw_doanh_thu│
└──────────────┘    READ       │  transform   │   WRITE                   └──────────────┘
                               └──────────────┘
                                      ▲
                               Task Scheduler
                               (6:00 AM daily)
```

### Ưu / nhược điểm

- Ưu: linh hoạt nhất, hỗ trợ mọi loại DB, transform mạnh bằng pandas
- Nhược: cần cài Python, thêm một layer quản lý ngoài SQL Server

---

## 2.3. ETL Tool (cho doanh nghiệp lớn)

### Tổng quan

Các tool ETL chuyên dụng có giao diện kéo thả, hỗ trợ connect nhiều loại DB.

| Tool | Loại | Chi phí |
|---|---|---|
| **Apache Airflow** | Open source | Miễn phí |
| **Talend Open Studio** | Open source | Miễn phí |
| **Azure Data Factory** | Cloud (Microsoft) | Trả phí theo dùng |
| **AWS Glue** | Cloud (Amazon) | Trả phí theo dùng |
| **Informatica** | Enterprise | Đắt |
| **SSIS** | Microsoft | Miễn phí (đi kèm SQL Server) |

Phù hợp khi: pipeline phức tạp, nhiều nguồn, cần monitoring/alerting, team lớn.

Không cần thiết khi: quy mô nhỏ, 2-3 nguồn, một mình quản lý → Python hoặc Linked Server là đủ.

---

# SO SÁNH TỔNG HỢP

## TH1: Cùng hệ quản trị (SQL Server A → SQL Server B)

| Tiêu chí | Linked Server + Agent | Replication | CDC |
|---|---|---|---|
| Độ khó | Dễ | Trung bình | Khó |
| Transform | Có (SQL) | Không | Có (SQL) |
| Tốc độ | Theo lịch | Gần realtime | Theo lịch |
| Incremental | Tự viết logic | Tự động | Tự động |
| Chiều data | Push hoặc Pull | A → B (Publisher → Subscriber) | A → B |
| Phù hợp | Logic đơn giản | Cần realtime | Data lớn, track thay đổi |

## TH2: Khác hệ quản trị (SQL Server → MySQL/PG/Mongo)

| Tiêu chí | Linked Server + ODBC | Python | ETL Tool |
|---|---|---|---|
| Độ khó | Trung bình | Dễ | Trung bình - Khó |
| Transform | Có (SQL, hạn chế) | Có (pandas, mạnh) | Có (giao diện) |
| Cross-platform | Cần driver riêng | Hỗ trợ mọi DB | Hỗ trợ mọi DB |
| Chi phí | Miễn phí | Miễn phí | Miễn phí - Đắt |
| Phù hợp | Query đơn giản | Quy mô nhỏ-vừa | Doanh nghiệp lớn |

## Khuyến nghị

- **Cùng SQL Server, đơn giản**: Linked Server + SQL Agent (Pull)
- **Cùng SQL Server, cần realtime**: Replication
- **Cùng SQL Server, data lớn**: CDC
- **Khác hệ quản trị, quy mô nhỏ-vừa**: Python
- **Khác hệ quản trị, doanh nghiệp lớn**: ETL Tool (Airflow, ADF)
- **Trường hợp của bạn (file + DB)**: Python kéo data vào raw, SQL xử lý transform
