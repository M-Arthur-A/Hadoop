### Логин на одну из машин в HADOOP

`ssh student8_9@89.208.221.132 -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa`

### Импорт таблицы из PostgreSQL в Hadoop с использованием SQOOP в HDFS в формате parquet (--as-parquetfile)

1. Получение перечня доступных баз данных:
   * `sqoop list-databases --connect jdbc:postgresql://node3.novalocal/pg_db --username exporter --password exporter_pass`
2. Получение доступных таблиц в базе данных pg_db
   * `sqoop list-tables --connect jdbc:postgresql://node3.novalocal/pg_db --username exporter --password exporter_pass`
3. Просмотр 10 строк выбранной таблицы
   * `sqoop eval -connect jdbc:postgresql://node3.novalocal/pg_db --username exporter --password exporter_pass --query "select * from character LIMIT 10"`
4. Просмотр типов данных выбранной таблицы для последующего создания таблицы в Hive
   * `sqoop eval -connect jdbc:postgresql://node3.novalocal/pg_db --username exporter --password exporter_pass --query "select table_name, column_name, data_type from information_schema.columns where table_name = 'character'"`
5. Импорт таблицы `character` в HDFS в формате PARQUET
   * `sqoop import --connect jdbc:postgresql://node3.novalocal/pg_db --username exporter --password exporter_pass --table character --target-dir /user/student8_9/sqoop --as-parquetfile`

### Построение поверх импортированных данных HIve таблицы

```sql
use student8_9__appstore;

create external table student8_9__appstore.shakespeare_characters(
	charid string,
	charname string,
	abbrev string,
	description string,
	speechcount int
)
stored as parquet
location '/user/student8_9/sqoop';

SELECT * FROM shakespeare_characters LIMIT 1000;
```

