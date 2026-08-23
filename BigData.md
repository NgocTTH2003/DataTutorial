# ⚡ ÔN TẬP NHANH BIG DATA
### Hadoop · Spark · Elasticsearch · Kafka

---

## 1. BIG DATA LÀ GÌ?

Dữ liệu quá lớn, quá nhanh, quá đa dạng để xử lý bằng công cụ truyền thống (Excel, MySQL đơn lẻ...).

**Mô hình 3V:**

| V | Ý nghĩa | Ví dụ |
|---|---|---|
| **Volume** | Khối lượng lớn | Facebook tạo 4 PB dữ liệu/ngày |
| **Velocity** | Tốc độ sinh ra nhanh | Log server, giao dịch ngân hàng mỗi giây |
| **Variety** | Đa dạng định dạng | Text, ảnh, video, JSON, log, sensor |

Mở rộng thêm: **Value** (giá trị) và **Veracity** (độ tin cậy).

---

## 2. HADOOP

### Hadoop là gì?
Framework mã nguồn mở để **lưu trữ và xử lý dữ liệu lớn** trên cluster gồm nhiều máy tính.

### 3 thành phần cốt lõi

```
┌─────────────────────────────────────┐
│            YARN                     │  ← Quản lý tài nguyên cluster
├─────────────────────────────────────┤
│   MapReduce        │   Spark (*)    │  ← Xử lý dữ liệu
├─────────────────────────────────────┤
│            HDFS                     │  ← Lưu trữ phân tán
└─────────────────────────────────────┘
```

**HDFS (Hadoop Distributed File System)**
- Chia file thành các block (mặc định **128 MB**)
- Mỗi block được sao chép **3 bản** trên các node khác nhau (replication factor = 3)
- Gồm **NameNode** (quản lý metadata) và **DataNode** (lưu dữ liệu thực)
- Thiết kế cho file lớn, ghi 1 lần đọc nhiều lần (write-once, read-many)

**MapReduce**
- Mô hình xử lý 2 bước: **Map** (chia nhỏ & xử lý song song) → **Reduce** (gom kết quả)
- Đọc/ghi đĩa ở mỗi bước → chậm
- Đang dần được thay thế bởi Spark

**YARN (Yet Another Resource Negotiator)**
- Quản lý CPU, RAM trên cluster
- Phân bổ tài nguyên cho các job (MapReduce, Spark, Hive...)
- Gồm **ResourceManager** (quản lý toàn cluster) và **NodeManager** (quản lý từng node)

### Các công cụ trong hệ sinh thái Hadoop

| Công cụ | Vai trò | Ghi nhớ nhanh |
|---|---|---|
| **Hive** | SQL trên Hadoop | Viết HiveQL → chuyển thành MapReduce job |
| **Pig** | Scripting xử lý dữ liệu | Pig Latin script, dễ hơn viết Java MapReduce |
| **HBase** | NoSQL database trên HDFS | Column-family, random read/write, real-time |
| **Sqoop** | Import/Export giữa RDBMS ↔ HDFS | SQL database ↔ Hadoop |
| **Flume** | Thu thập log vào HDFS | Log server → HDFS |
| **Zookeeper** | Điều phối cluster | Quản lý config, đồng bộ, leader election |
| **Oozie** | Lập lịch workflow | Chạy job theo thứ tự, theo lịch |

---

## 3. APACHE SPARK

### Spark là gì?
Engine xử lý dữ liệu lớn **in-memory** (trong bộ nhớ), nhanh hơn MapReduce 10-100x.

### Spark vs MapReduce

| | MapReduce | Spark |
|---|---|---|
| Xử lý | Đọc/ghi đĩa mỗi bước | In-memory |
| Tốc độ | Chậm | Nhanh 10-100x |
| Ngôn ngữ | Java | Java, Scala, **Python (PySpark)**, R |
| Xử lý | Chỉ batch | Batch + Streaming + ML + Graph |
| Dễ dùng | Khó (viết nhiều code Java) | Dễ (API cấp cao, SQL) |

### 4 module của Spark

```
┌──────────┬──────────────┬─────────┬──────────┐
│ Spark SQL│Spark Streaming│  MLlib  │  GraphX  │
│ (SQL,    │ (real-time   │ (machine│ (graph   │
│  batch)  │  streaming)  │ learning│processing│
├──────────┴──────────────┴─────────┴──────────┤
│              Spark Core (RDD)                │
└──────────────────────────────────────────────┘
```

### Khái niệm quan trọng

**RDD (Resilient Distributed Dataset)**
- Cấu trúc dữ liệu cơ bản, **bất biến** (immutable), phân tán trên cluster
- Hỗ trợ 2 loại thao tác: **Transformation** (tạo RDD mới: map, filter) và **Action** (trả kết quả: count, collect)
- Fault tolerance qua **lineage** (ghi nhớ cách tạo ra, có thể tính lại nếu mất)

**DataFrame**
- Như bảng trong SQL, có tên cột và kiểu dữ liệu
- Tối ưu hơn RDD nhờ Catalyst optimizer
- API chính được dùng hiện nay (thay vì RDD trực tiếp)

**PySpark**
- Python API cho Spark
- Viết Spark bằng Python → phổ biến với DA/DE
- Ví dụ đơn giản:
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("demo").getOrCreate()
df = spark.read.csv("data.csv", header=True)
df.filter(df["age"] > 25).groupBy("city").count().show()
```

**Spark SQL**
- Chạy SQL trên DataFrame
- Đọc được: JSON, CSV, Parquet, Hive, JDBC (MySQL, PostgreSQL...)
```python
df.createOrReplaceTempView("users")
spark.sql("SELECT city, COUNT(*) FROM users WHERE age > 25 GROUP BY city").show()
```

---

## 4. ELASTICSEARCH

### Elasticsearch là gì?
Search engine phân tán, mã nguồn mở. Tìm kiếm full-text và phân tích dữ liệu **gần real-time**.

### ELK Stack

```
Data Sources → Logstash → Elasticsearch → Kibana
(log, API...)   (thu thập,   (lưu trữ,      (trực quan,
                 parse,       index,          dashboard)
                 transform)   tìm kiếm)
```

| Thành phần | Vai trò |
|---|---|
| **Elasticsearch** | Lưu trữ, index và tìm kiếm dữ liệu (JSON documents) |
| **Logstash** | Thu thập, parse, transform dữ liệu từ nhiều nguồn |
| **Kibana** | Trực quan hóa, tạo dashboard, monitor |
| **Beats** (bổ sung) | Agent nhẹ thu thập data (Filebeat, Metricbeat...) |

### So sánh thuật ngữ ES vs RDBMS

| Elasticsearch | RDBMS |
|---|---|
| Index | Database / Table |
| Document | Row |
| Field | Column |
| Mapping | Schema |
| Shard | Partition |

### Đặc điểm chính
- Dữ liệu lưu dạng **JSON document**
- Tìm kiếm **full-text** cực nhanh (inverted index)
- **Near real-time**: document mới có thể tìm thấy trong ~1 giây
- **Phân tán**: tự động chia shard, replicate trên cluster
- RESTful API: tương tác qua HTTP GET/POST/PUT/DELETE

### Ứng dụng phổ biến
- Log management & monitoring (DevOps)
- Search engine cho website/app
- Business analytics & reporting
- Security analytics (SIEM)

---

## 5. APACHE KAFKA

### Kafka là gì?
Nền tảng **streaming phân tán**, cho phép truyền tải message giữa các hệ thống với throughput cao, độ trễ thấp.

### Kiến trúc Kafka

```
Producer ──→ Topic (Partition 0) ──→ Consumer Group
             Topic (Partition 1) ──→ Consumer Group
             Topic (Partition 2) ──→ Consumer Group
                    │
                  Broker (Kafka Server)
```

### Thuật ngữ cần biết

| Thuật ngữ | Ý nghĩa |
|---|---|
| **Producer** | Gửi message vào topic |
| **Consumer** | Đọc message từ topic |
| **Topic** | Kênh/danh mục chứa message |
| **Partition** | Chia nhỏ topic → xử lý song song |
| **Broker** | Server Kafka, lưu trữ partition |
| **Consumer Group** | Nhóm consumer cùng đọc 1 topic (mỗi partition chỉ 1 consumer trong group đọc) |
| **Offset** | Vị trí message trong partition (đánh số thứ tự) |

### Đặc điểm
- **Throughput cao**: hàng triệu message/giây
- **Durable**: message được lưu trên đĩa, không mất khi consumer chưa đọc
- **Scalable**: thêm broker/partition dễ dàng
- **Pub/Sub model**: producer và consumer tách rời (decoupled)

### Ứng dụng
- Pipeline dữ liệu real-time (log → Kafka → Elasticsearch)
- Event-driven architecture
- Đồng bộ dữ liệu giữa microservices
- CDC (Change Data Capture) từ database

---

## 6. SO SÁNH TỔNG THỂ

### Khi nào dùng công cụ nào?

| Nhu cầu | Công cụ |
|---|---|
| Lưu trữ file lớn phân tán | **HDFS** |
| Xử lý batch dữ liệu lớn | **Spark** (hoặc MapReduce) |
| Xử lý streaming real-time | **Spark Streaming** + **Kafka** |
| Truy vấn SQL trên dữ liệu lớn | **Hive** hoặc **Spark SQL** |
| Tìm kiếm full-text & log | **Elasticsearch** |
| Truyền tải message real-time | **Kafka** |
| Dashboard monitoring | **Kibana** |
| ETL pipeline | **Spark** + **Kafka** + **Airflow** |

### Data Lake vs Data Warehouse

| | Data Lake | Data Warehouse |
|---|---|---|
| Dữ liệu | Thô (raw), mọi định dạng | Đã xử lý, có cấu trúc |
| Schema | Schema-on-read | Schema-on-write |
| Người dùng | Data Engineer, Data Scientist | Business Analyst, DA |
| Công cụ | Hadoop, Spark, S3 | Redshift, BigQuery, Snowflake |
| Mục đích | Khám phá, ML, lưu trữ tổng | Reporting, BI, dashboard |

### Batch vs Stream Processing

| | Batch | Stream |
|---|---|---|
| Xử lý | Theo lô, định kỳ | Liên tục, từng event |
| Độ trễ | Phút → giờ | Mili giây → giây |
| Ví dụ | Báo cáo cuối ngày | Phát hiện gian lận real-time |
| Công cụ | MapReduce, Spark batch | Kafka, Spark Streaming, Flink |

---

## 7. CÂU HỎI PHỎNG VẤN HAY GẶP

**Q: HDFS lưu trữ dữ liệu như thế nào?**
→ Chia file thành block 128MB, replicate 3 bản trên các DataNode khác nhau. NameNode quản lý metadata.

**Q: Tại sao Spark nhanh hơn MapReduce?**
→ Spark xử lý in-memory, MapReduce đọc/ghi đĩa mỗi bước. Spark cũng tối ưu DAG (Directed Acyclic Graph) thay vì chạy tuần tự.

**Q: RDD vs DataFrame?**
→ RDD: API cấp thấp, linh hoạt, type-safe. DataFrame: API cấp cao, có schema, tối ưu bởi Catalyst → nhanh hơn, dễ dùng hơn, khuyến khích dùng.

**Q: Khi nào dùng Elasticsearch thay vì SQL database?**
→ Khi cần full-text search, log analytics, aggregation trên dữ liệu lớn với tốc độ gần real-time. SQL database tốt hơn cho CRUD transaction.

**Q: Kafka khác message queue truyền thống (RabbitMQ) ở đâu?**
→ Kafka lưu message trên đĩa (durable), replay được, throughput cao hơn nhiều, thiết kế cho Big Data streaming. RabbitMQ phù hợp task queue nhỏ hơn.

**Q: Data Lake vs Data Warehouse?**
→ Lake: dữ liệu thô, schema-on-read, cho DE/DS khám phá. Warehouse: dữ liệu sạch, schema-on-write, cho BA/DA làm report.

---

> 💡 **Mẹo ôn thi**: Tập trung nhớ **mỗi công cụ dùng để làm gì** và **khi nào chọn công cụ nào**. Không cần nhớ chi tiết command hay config — đó là việc tra docs khi làm thực tế.
