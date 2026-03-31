# SQL Usage

---

## Overview

---

本文档主要用于记录 SQL 语法，参考使用的数据库为 DRWDB 。

## 语法

---

###  1. 查询

- 1. 查询所有数据

```sql
SELECT * 
FROM sensor_data ;
```

- 2. 查询指定列 

```sql
SElECT id 
FROM sensor_data ;
```

- 3. 条件查询
  
```sql
-- 条件查询，显示一行数据
SELECT *
FROM sensor_data 
WHERE id = 1 ;

-- 多条件查询
SELECT *
FROM sensor_data
WHERE temperature > 20 AND humidity < 70;

-- 排序
SELECT *
FROM sensor_data
ORDER BY timestamp DESC;  -- DESC表示降序排序，ASC表示升序排序

-- 查询指定数量的最新数据
SELECT *
FROM sensor_data
ORDER BY timestamp DESC
LIMIT 5;
```
### 2. 统计分析

- 1. 计数
```sql
SELECT COUNT(*)
FROM sensor_data;
```

- 2. 平均值，最值
```sql
SELECT MAX(temperature) --MAX，MIN，AVG
FROM sensor_data;
```

- 3. 分组
  
```sql
SELECT device_id, AVG(temperature)
FROM sensor_data
GROUP BY device_id;
```
### 3. 插入数据

```sql
INSERT INTO sensor_data
(device_id, timestamp, temperature, humidity, pressure, voltage)
VALUES
('device001', NOW(), 23.5, 60.1, 1012.3, 3.7); --NOW()传入现在时间
```

### 4. 更新数据
```sql
UPDATE sensor_data
SET temperature = 25
WHERE id = 1;
-- 注意：update语句需要指定条件，否则会更新所有数据

-----------------------------delete------------------
DELETE FROM sensor_data
WHERE id = 1;
-- 注意：一定要加筛选条件，否则就会删除所有数据
```

### 5. 表结构操作

- 1. 查看表
```sql
SHOW TABLES；
```

- 2. 查看表结构

```sql
DESCRIBE sensor_data;
```

- 3. 创建表

```sql
CREATE TABLE sensor_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    device_id VARCHAR(50),
    timestamp DATETIME,
    temperature FLOAT,
    humidity FLOAT,
    pressure FLOAT,
    voltage FLOAT
);
```

- 4. 修改表结构

```sql  
ALTER TABLE sensor_data
ADD battery FLOAT;

ALTER TABLE sensor_data
DROP COLUMN battery;

-- 5.删除表
DROP TABLE sensor_data；
```

### 语法执行顺序

- 1. FROM
- 2. WHERE
- 3. GROUP BY
- 4. HAVING
- 5. SELECT
- 6. ORDER BY
- 7. LIMIT