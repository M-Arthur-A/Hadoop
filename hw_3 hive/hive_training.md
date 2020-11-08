**Копируем датасет на удаленный сервер**

`scp -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/3_dataset/appstore_games.csv student8_9@89.208.221.132:/home/student8_9/ `

**Копируем файл с удаленного серевера в HDFS**

`hdfs dfs -copyFromLocal appstore_games.csv /test_arthur_dir`

`hdfs dfs -cp /test_arthur_dir/appstore_games.csv /user/student8_9/`

**Проверяем есть ли файл в hdfs**

`hdfs dfs -ls /user/student8_9/`

**Далее заходим в HUE для создания и работы с hive БД**

http://89.208.221.132:8888/hue/accounts/login/?next=/hue/

**Делаем запросы**

```sql
-- создание БД
CREATE DATABASE student8_9__appstore;

-- создание таблицы
drop table student8_9__appstore.app_stat;
create external table student8_9__appstore.app_stat
(
URL string,
ID int,
Name string,
Subtitle string,
Icon_URL string,
Average_User_Rating float,
User_Rating_Count int,
Price float,
in_app_Purchases string,
Description string,
Developer string,
Age_Rating string,
Languages string,
size_ int,
Primary_Genre string,
Genres string,
Original_Release_Date string,
Current_Version_Release_Date string
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
TBLPROPERTIES (
                'serialization.null.format' = '',
                'skip.header.line.count' = '1'
                )
;

-- загрузка данных из файла
LOAD DATA INPATH '/user/student8_9/appstore_games.csv' INTO TABLE student8_9__appstore.app_stat;

-- проверка наличия данных
SELECT * FROM app_stat;

-- запрос на распределение приложений по рейтингам
select a_s.average_user_rating, COUNT(*) from app_stat a_s group by a_s.average_user_rating;
-- вывод:
-- -	9446
-- 1	14
-- 1.5	60
-- 2	158
-- 2.5	317
-- 3	514
-- 3.5	925
-- 4	1722
-- 4.5	2861
-- 5	990

-- тест JOIN'а
SELECT a_s.name, a_ss.genres 
	from app_stat a_s 
	JOIN app_stat a_ss 
		on a_ss.id = a_s.id 
		WHERE a_ss.average_user_rating = '4.5' AND a_s.user_rating_count > 10000  
	LIMIT 50;
```

