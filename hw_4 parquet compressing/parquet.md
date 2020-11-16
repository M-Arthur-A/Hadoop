### Создание тиаблиц в формате parquet с сжатиеем и без
```sql
use student8_9__appstore;

-- Дана большая таблица
-- Её размер вот такой:
-- hdfs dfs -du -h -s /test_datasets/citation
-- 97.2 G 291.5 G /test_datasets/citation


-- 1. Создать две таблицы в форматах PARQUET/ORC/AVRO c компрессией и без оной.
-- a. Первая без компрессии в произвольном формате (выберите один из форматов хранения)
create external table citation_data
(
oci string,
citing string,
cited string,
creation string,
timespan string,
journal_sc string,
author_sc string
)
STORED as PARQUET;
-- b. Вторая с компрессией в том-же формате. (выберите произвольный способ сжатия из поддерживаемых)
create external table citation_data_snappy
(
oci string,
citing string,
cited string,
creation string,
timespan string,
journal_sc string,
author_sc string
)
STORED as PARQUET
TBLPROPERTIES ("parquet.compression"="SNAPPY");

SET hive.exec.compress.output=true;
-- 2. Заполнить данными из большой таблицы hive_db.citation_data
INSERT INTO TABLE student8_9__appstore.citation_data 
SELECT * FROM hive_db.citation_data LIMIT 10000;

INSERT INTO TABLE student8_9__appstore.citation_data_snappy 
SELECT * FROM hive_db.citation_data LIMIT 10000;
```

### Верифицировать способ сжатия при помощи parquet-tools/avro-tools/hive --orcfiledump

`[student8_9@manager ~]$ parquet-tools meta hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data/000000_0`

```plain
oci:          BINARY UNCOMPRESSED DO:0 FPO:4 SZ:785645/785645/1,00 VC:10000 [more]...
citing:       BINARY UNCOMPRESSED DO:0 FPO:785649 SZ:5563/5563/1,00 VC:10000 [more]...
cited:        BINARY UNCOMPRESSED DO:0 FPO:791212 SZ:258391/258391/1,00 VC:10000 [more]...
creation:     BINARY UNCOMPRESSED DO:0 FPO:1049603 SZ:263/263/1,00 VC:10000 [more]...
timespan:     BINARY UNCOMPRESSED DO:0 FPO:1049866 SZ:9708/9708/1,00 VC:10000 [more]...
journal_sc:   BINARY UNCOMPRESSED DO:0 FPO:1059574 SZ:697/697/1,00 VC:10000 [more]...
author_sc:    BINARY UNCOMPRESSED DO:0 FPO:1060271 SZ:62/62/1,00 VC:10000 [more]...
```

`[student8_9@manager ~]$ parquet-tools meta hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data_snappy/000000_0`

```plain
oci:          BINARY SNAPPY DO:0 FPO:4 SZ:206820/792539/3,83 VC:10000 ENC:RLE,PLAIN [more]...
citing:       BINARY SNAPPY DO:0 FPO:206824 SZ:2096/5949/2,84 VC:10000 ENC:RLE,PLAI [more]...
cited:        BINARY SNAPPY DO:0 FPO:208920 SZ:134742/262610/1,95 VC:10000 ENC:RLE, [more]...
creation:     BINARY SNAPPY DO:0 FPO:343662 SZ:233/227/0,97 VC:10000 ENC:RLE,PLAIN_ [more]...
timespan:     BINARY SNAPPY DO:0 FPO:343895 SZ:9195/9548/1,04 VC:10000 ENC:RLE,PLAI [more]...
journal_sc:   BINARY SNAPPY DO:0 FPO:353090 SZ:366/387/1,06 VC:10000 ENC:RLE,PLAIN_ [more]...
author_sc:    BINARY SNAPPY DO:0 FPO:353456 SZ:66/62/0,94 VC:10000 ENC:RLE,PLAIN_DI [more]...
```



### Посмотреть на получившийся размер данных.

`hdfs dfs -ls hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data/000000_0`

```plain
-rwxrwxrwx   3 student8_9 hive    1061152 2020-11-16 19:48 hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data/000000_0
```

`hdfs dfs -ls hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data_snappy/000000_0`

```plain
-rwxrwxrwx   3 student8_9 hive     354381 2020-11-16 19:56 hdfs://manager.novalocal:8020/user/hive/warehouse/student8_9__appstore.db/citation_data_snappy/000000_0
```

Получились следующие размеры таблиц:

| без компрессии | с SNAPPY компрессией |
| -------------- | -------------------- |
| 1 061 152      | 354 381              |



### Сделать выводы о эффективности хранения и сжатия.

Таким образом, **сжатие на 66%**.