**Копируем датасет на удаленный сервер**

`scp -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/3_dataset/appstore_games.csv student8_9@89.208.221.132:/home/student8_9/ `

**Копируем файл с удаленного серевера в HDFS**

`hdfs dfs -copyFromLocal appstore_games.csv /test_arthur_dir`

`hdfs dfs -cp /test_arthur_dir/appstore_games.csv /user/student8_9/`

**Проверяем есть ли файл в hdfs**

`hdfs dfs -ls /user/student8_9/`

**Далее заходим в HUE для создания и работы с hive БД**

http://89.208.221.132:8888/hue/accounts/login/?next=/hue/