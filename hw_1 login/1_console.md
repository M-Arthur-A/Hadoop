**Логин**

```ssh student8_9@89.208.221.132 -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa```

**Создаем папку**

```hdfs dfs -mkdir /test_arthur_dir```

Проверяем

`hdfs dfs -ls /`

`hdfs dfs -ls / |grep arthur`

**Копируем файл на удаленный сервер**

`scp -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/1_start/help_href.txt student8_9@89.208.221.132:/home/student8_9/`

**Копируем файл с удаленного серевера в HDFS**

`hdfs dfs -copyFromLocal help_href.txt /test_arthur_dir`

**Создаем вложенную папку и копируем туда файл, провяем**

`hdfs dfs -mkdir /test_arthur_dir/help_dir`

`hdfs dfs -cp /test_arthur_dir/help_href.txt /test_arthur_dir/help_dir/help_href.txt`

`hdfs dfs -ls /test_arthur_dir/help_dir`

**Проверяем сколько весит папка и файл**

`hdfs dfs -du /test_arthur_dir`

**Смотрим как устроено хранение папки**

`hdfs fsck /test_arthur_dir`

**Устанвливаем нестандартное число репликаций для файла**

`hdfs dfs -setrep 2 /test_arthur_dir/help_href.txt`