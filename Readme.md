# Docker Compose
sudo sysctl -w vm.max_map_count=2000000

# Flink SQL

```SQL
CREATE CATALOG catalog WITH (
    'type' = 'paimon',
    'metastore' = 'hive',
    'uri' = 'thrift://hive-metastore:9083',
    'warehouse' = 's3://datalake/paimon',
    's3.endpoint' = 'http://minio:9000',
    's3.access-key' = 'minio',  
    's3.secret-key' = 'minio123',
    's3.path.style.access' = 'true',
    'location-in-properties' = 'true'
);

CREATE CATALOG paimon WITH (
    'type' = 'paimon',
    'warehouse' = 's3://datalake/paimon',
    's3.endpoint' = 'http://minio:9000',
    's3.access-key' = 'minio',  
    's3.secret-key' = 'minio123',
    's3.path.style.access' = 'true',
    'location-in-properties' = 'true'
);

USE CATALOG catalog;

CREATE DATABASE medallion;

USE medallion;

CREATE TABLE bronze (
    event_id STRING,
    user_id STRING,
    event_type STRING,
    event_timestamp TIMESTAMP(3),
    raw_data STRING,
    processing_time AS PROCTIME()
);

CREATE TABLE silver (
    event_id STRING,
    user_id STRING,
    event_type STRING,
    event_timestamp TIMESTAMP(3),
    raw_data STRING
);

SET 'execution.checkpointing.interval' = '1 s';

INSERT INTO silver SELECT event_id, user_id, event_type, event_timestamp, raw_data  FROM bronze;

INSERT INTO bronze (event_id, user_id, event_type, event_timestamp, raw_data)
VALUES 
    ('e1', 'user1', 'login', CURRENT_TIMESTAMP, '{"device": "mobile"}'),
    ('e2', 'user1', 'click', CURRENT_TIMESTAMP, '{"page": "home"}'),
    ('e3', 'user2', 'login', CURRENT_TIMESTAMP, '{"device": "desktop"}');

SELECT * FROM bronze;
SELECT * FROM silver;
```

# Doris SQL
mysql -h 127.0.0.1 -P 9030 -u root

SET enable_file_cache = false;

CREATE CATALOG paimon PROPERTIES (
    "type" = "paimon",
    "warehouse" = "s3://datalake/paimon",
    "s3.endpoint" = "http://172.20.0.12:9000",
    "s3.access_key" = "minio",
    "s3.secret_key" = "minio123"
);