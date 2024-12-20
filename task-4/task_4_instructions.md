# Гайд по выполнению 4 задания

## Скачиваем и настраиваем Spark

### 1. Подключение к Jump Node
   ```bash
   ssh team-19-nn
   ```
### 2. Скачиваем дистрибутив Spark
   ```bash
   wget -q "https://dlcdn.apache.org/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz"
   ``` 
### 3. Разархивируем дистрибутив Spark
   ```bash
   tar -xzf "spark-3.5.3-bin-hadoop3.tgz"
   ``` 
### 4. Скачиваем дистрибутив Hive (для совместимости с версией)
   ```bash
   wget https://archive.apache.org/dist/hive/hive-4.0.0-alpha-2/apache-hive-4.0.0-alpha-2-bin.tar.gz
   ``` 
### 5. Разархивируем дистрибутив Hive
   ```bash
   tar -xzf apache-hive-4.0.0-alpha-2-bin.tar.gz
   ``` 
### 6. Добавляем переменные окружения в конфиг ~/.profile
   ```bash
export HIVE_HOME=/home/hadoop/apache-hive-4.0.0-alpha-2-bin
export PATH=$HIVE_HOME/bin:$PATH
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_HOME=/home/hadoop/spark-3.5.3-bin-hadoop3
export PATH=$PATH:$SPARK_HOME/bin
export YARN_CONF_DIR=/home/hadoop/hadoop-3.4.0/etc/hadoop

   ```  


### 7. Активируем переменные окружения в конфиге ~/.profile
   ```bash
   source ~/.profile
   ```
### 8. Передадим файл с jn с данными на nn в юзера hadoop
   ```bash
   scp organizations-2000000.csv team-19-nn:/home/hadoop/
   ```

### 9. Запустим в отдельном терминале инстанс hive
   ```bash
    hive
    --hiveconf hive.server2.enable.doAs=false
    --hiveconf hive.security.authorization.enabled=false
    --service metastore
    1>> /tmp/metastore.log
    2>> /tmp/metastore.log
   ``` 

### 10. Запустим в фоне metastore
   ```bash
   hive --service metastore -p 5433 & 
   ```    

### 10. В первом терминале переключаемся на sudo юзера team
   ```bash
    ssh team-19-nn
    su team 
   ```

### 11. Установка pyspark в отдельном окружении
   ```bash
    sudo apt install python3.12-venv
    sudo -i -u hadoop
    cd ~
    source ~/.profile
    python3 -m venv .venv
    source .venv/bin/activate
    pip install pyspark
   ``` 
### 12. Запускаем python
   ```bash
    python
   ``` 
### 13. Считаем наши данные с помощью spark и обрабатываем их (заменить ip_namenode на действительный namenode ip)
   ```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
spark = SparkSession.builder \
.master("yarn") \
.appName("spark-with-yarn") \
.config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
.config("spark.hadoop.hive.metastore.uris", "thrift://ip_namenode:5433") \
.enableHiveSupport() \
.getOrCreate()
    

df = spark.read.csv(
    "/input/organizations-2000000.csv",
    header=True,
    inferSchema=True
)

print(df.columns)

transformed_df = df.select([
    F.max(df.Founded),
    F.min(df.Index),
    F.mean(df['Number of employees']),
    F.count_distinct(df.Country),
    F.count_distinct(df.Name)
])

transformed_df.show()

print(f"Current number partitions: {df.rdd.getNumPartitions()}") # 3

df = df.repartition(2, "Founded")

print(f"Current number of partitions: {df.rdd.getNumPartitions()}") # 2

df.write.parquet("/out/organizations") # сохраняем в виде файла на hdfs
df.write.saveAsTable("test_df", partitionBy="Founded") # сохраняем в виде таблицы в hive
   ``` 



