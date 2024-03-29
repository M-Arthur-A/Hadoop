## Создаем flume в cloudera manager

Переход в Cloudera Manager http://89.208.221.132:7180/cmf/home

Задаем наименование flume: flume8_9 на [node2.novalocal](http://89.208.221.132:7180/cmf/hardware/hosts/5/status)

Конфигурируем следующим образом: добавляя в файл на локальной машине `/tmp/student8_9_to_flume.txt` строки, flume их сохраняет в файловую систему HDFS в формате avro `/user/student8_9/flume=[время]/` и в БД HBase (2 sink)

```flume
Flume8_9.sources = SourceExec
Flume8_9.channels = MemChannel
Flume8_9.sinks = HDFSsink HBase

# Источник
Flume8_9.sources.SourceExec.interceptors = ts_interceptor
Flume8_9.sources.SourceExec.interceptors.ts_interceptor.type = timestamp
Flume8_9.sources.SourceExec.type = exec
Flume8_9.sources.SourceExec.command =  tail -f /tmp/student8_9_to_flume.txt

# Описание канала
Flume8_9.channels.MemChannel.type = memory 
Flume8_9.channels.MemChannel.capacity = 1000 
Flume8_9.channels.MemChannel.transactionCapacity = 10

# Описание синка
Flume8_9.sinks.HDFSsink.type = hdfs
Flume8_9.sinks.HDFSsink.hdfs.filePrefix = Flume_8_9_data
Flume8_9.sinks.HDFSsink.hdfs.path = /user/student8_9/flume=%Y%m%d/
Flume8_9.sinks.HDFSsink.hdfs.fileType = DataStream
Flume8_9.sinks.HDFSsink.hdfs.rollSize = 1048576
Flume8_9.sinks.HDFSsink.hdfs.rollCount = 0
Flume8_9.sinks.HDFSsink.hdfs.fileSuffix = .avro
Flume8_9.sinks.HDFSsink.serializer = avro_event
#Flume8_9.sinks.HDFSsink.serializer.compressionCodec = snappy

Flume8_9.sinks.HBase.type = hbase
Flume8_9.sinks.HBase.table = student8_9_hbase
Flume8_9.sinks.HBase.columnFamily = logs
Flume8_9.sinks.HBase.serializer = org.apache.flume.sink.hbase.RegexHbaseEventSerializer

# Связь трех компонентов между собой
Flume8_9.sources.SourceExec.channels = MemChannel
Flume8_9.sinks.HDFSsink.channel = MemChannel
Flume8_9.sinks.HBase.channel = MemChannel
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

## Создание таблицы в Hbase

` create 'student8_9_hbase', 'Name', 'logs'` – создание таблицы student8_9_hbase с id строк Name и группой солбцов lo

`list` - проверяем создалась ли  таблица

