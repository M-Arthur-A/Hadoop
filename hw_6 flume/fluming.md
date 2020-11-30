## Создаем flume в cloudera manager

Переход в Cloudera Manager http://89.208.221.132:7180/cmf/home

Задаем наименование flume: flume8_9 на [node2.novalocal](http://89.208.221.132:7180/cmf/hardware/hosts/5/status)

Конфигурируем следующим образом: добавляя в файл `/user/student8_9/to_flume.txt` строки, flume их сохраняет в формате avro `/user/student8_9/flume/`

```flume
Flume8_9.sources = SourceExec
Flume8_9.channels = MemChannel
Flume8_9.sinks = HDFSsink

# Источник
Flume8_9.sources.SourceExec.type = exec
Flume8_9.sources.SourceExec.command =  tail -F /user/student8_9/to_flume.txt


# Описание канала
Flume8_9.channels.MemChannel.type = memory 
Flume8_9.channels.MemChannel.capacity = 1000 
Flume8_9.channels.MemChannel.transactionCapacity = 10

# Описание синка
Flume8_9.sinks.HDFSsink.type = hdfs
Flume8_9.sinks.HDFSsink.hdfs.filePrefix = Flume_8_9_data
Flume8_9.sinks.HDFSsink.hdfs.path = /user/student8_9/flume/
Flume8_9.sinks.HDFSsink.hdfs.fileType = SequenceFile
Flume8_9.sinks.HDFSsink.hdfs.fileSuffix = .avro
Flume8_9.sinks.HDFSsink.serializer = avro_event
Flume8_9.sinks.HDFSsink.serializer.compressionCodec = snappy

# Связь трех компонентов между собой
Flume8_9.sources.SourceExec.channels = MemChannel
Flume8_9.sinks.HDFSsink.channel = MemChannel
```

## Тест работы flume

Логин по ssh:  `ssh student8_9@89.208.221.132 -i /home/arthur/Project/GeekBrains/Hadoop/\!ADDS/id_rsa`

Создаем файл, в который будем дописывать строки: `hdfs dfs -touchz /user/student8_9/to_flume.txt`

Дописываем строку в этот файл: `echo "1st test row" | hadoop fs -appendToFile - /user/student8_9/to_flume.txt`

Читаем файл: `hadoop fs -cat /user/student8_9/to_flume.txt`

Добавляем еще строки: `echo "2nd test row" | hadoop fs -appendToFile - /user/student8_9/to_flume.txt`

`echo "3d test row" | hadoop fs -appendToFile - /user/student8_9/to_flume.txt` x10

`hdfs dfs -ls /user/student8_9/flume`

## На базе созданного flume avro-файла конфигурация таблицы hive

Открытие hive в hue http://89.208.221.132:8888/hue/editor/?type=hive

