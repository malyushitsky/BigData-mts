# Гайд по выполнению 3 задания

## Настройка PostgreSQL

### 1. Подключение к Name Node
```bash
ssh team-19-nn
```

### 2. Установка PostgreSQL
```bash
sudo apt install postgresql
```

### 3. Переключение на пользователя postgres
```bash
sudo -i -u postgres
```

### 4. Подключение к консоли PostgreSQL
```bash
psql
```

### 5. Создание бд
```sql
CREATE DATABASE metastore;
```

### 6. Создание пользователя (hive) для бд
```sql
CREATE USER hive WITH PASSWORD 'your_password';
```

### 7. Предоставление прав на бд
```sql
GRANT ALL PRIVILEGES ON DATABASE "metastore" TO hive;
```

### 8. Передача прав владения бд нашему пользователю
```sql
ALTER DATABASE metastore OWNER TO hive;
```

### 9. Выход из консоли PostgreSQL
```sql
\q
```

### 10. Выход из пользователя postgres на Name Node
 ```bash
 exit
 ```

### 11. Правка конфигурационных файлов PostgreSQL
#### 11.1 Открытие файла `postgresql.conf`:
```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```
#### 11.2 В секции "CONNECTIONS AND AUTHENTICATION" в разделе Connection settings добавляем адрес Name Node:
```plaintext
listen_addresses = 'team-19-nn'
```
    
#### 11.3 Открытие файла `pg_hba.conf`:
```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```
#### 11.4 В секции "IPv4 local connections" в начале добавляем:
```plaintext
host    metastore       hive            <ip_jumpnode>/32         password
```
#### 11.5 В той же секции удаляем строку:
```plaintext
host    all             all             127.0.0.1/32            scram-sha-256
```

### 12. Перезапуск PostgreSQL
```bash
sudo systemctl restart postgresql
```

### 13. Проверка статуса PostgreSQL
```bash
sudo systemctl status postgresql
```

### 14. Переход на Jump Node
```bash
exit
```

### 15. Установка PostgreSQL Client
```bash
sudo apt install postgresql-client-16
```

### 16. Проверяем возможность подключения к таблице
```bash
psql -h team-19-nn -p 5432 -U hive -W -d metastore
```

### 17. Выход из консоли PostgreSQL
```sql
\q
```

## Настройка Hadoop на Jump Node
### 1. Переключение на пользователя hadoop
```bash
sudo -i -u hadoop
```

### 2. Разархивация архива с Hadoop (внутри пользователя хадуп в корень файловой системы, архив дистрибутива скачивался ранее в task-1)
```bash
tar -xvzf hadoop-3.4.0.tar.gz
```

### 3. Добавление переменных окружения
#### 3.1 Открытие нужного файла
```bash
nano ~/.profile
```
#### 3.2 Добавление в конец файла следующих строки для настройки окружения Hadoop
```plaintext
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

### 4. После редактирования файла применяем изменения
```bash
source ~/.profile
```

### 5. Проверка установки Hadoop
```bash
hadoop version
```

### 6. Переход в папку дистрибутива хадуп
```bash
cd hadoop-3.4.0/etc/hadoop
```

### 7. Редактирование конфигурационного файла _core-site.xml_
```bash
nano core-site.xml
```
Добавляем несколько строк в файл:
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://team-19-nn:9000</value>
    </property>
</configuration>
```

### 8. Редактирование конфигурационного файла _hdfs-site.xml_
#### 8.1 Открытие файла
```bash
nano hdfs-site.xml
```
#### 8.1 Добавление строк в файл
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>
```

## Настройка Apache Hive

### 1. Переход в корень файловой системы
 ```bash
 cd 
 ```

### 2. Скачивание дистрибутива Apache Hive
```bash
wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz
```

### 3. Разархивация архива 
```bash
tar -xvzf apache-hive-4.0.1-bin.tar.gz
```

### 4. Переход в папку дистрибутива хайв
```bash
cd apache-hive-4.0.1-bin/
```

### 5. Проверка отсутствия нужного драйвера
```bash
cd lib
ls -l | grep postgres
```

### 6. Скачивание драйвера
```bash
wget https://jdbc.postgresql.org/download/postgresql-42.7.4.jar
```


### 7. Настройка конфигурационных файлов Hive
#### 7.1 Переход в нужную директорию:
```bash
cd ../conf/
```
#### 7.2 Создание нового файла `hive-site.xml`:
```bash
nano hive-site.xml
```
Вставляем:
```xml
<configuration>
   <property>
       <name>hive.server2.authentication</name>
       <value>NONE</value>
   </property>
   <property>
       <name>hive.metastore.warehouse.dir</name>
       <value>/user/hive/warehouse</value>
   </property>
   <property>
       <name>hive.server2.thrift.port</name>
       <value>5433</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionURL</name>
       <value>jdbc:postgresql://team-19-nn:5432/metastore</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionDriverName</name>
       <value>org.postgresql.Driver</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionUserName</name>
       <value>hive</value>
   </property>
   <property>
       <name>javax.jdo.option.ConnectionPassword</name>
       <value><postgre password></value>
   </property>
</configuration>
```

### 8. Редактирование .profile для Hive
#### 8.1 Открытие файла
```bash
nano ~/.profile
```
#### 8.2 Добавление путей в файл
```plaintext
export HIVE_HOME=/home/hadoop/apache-hive-4.0.1-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin
```

### 9. Активация нашего окружение
```bash
source ~/.profile
```

### 10. Проверка версий Hive и Hadoop
```bash
hive --version
hadoop version
```

### 11. Создание директории HDFS (предварительно с помощью webUI проверить отсутствие таких папок)
```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -chmod g+w /tmp
hdfs dfs -chmod g+w /user/hive/warehouse
```

### 12. Инициализация схемы Hive
```bash
cd ../
bin/schematool -dbType postgres -initSchema
```

### 13. Запуск Hive Server
```bash
hive --hiveconf hive.server2.enable.doAs=false --hiveconf hive.security.authorization.enabled=false --service hiveserver2 1>> /tmp/hs2.log 2>> /tmp/hs2.log &
```

### 14. Проверка запуска hive
```bash
jps
```

##  Подключение к Hive через Билайн

### 1. Активация окружения
```bash
source ~/.profile
```

### 2. Подключение к Hive
```bash
beeline -u jdbc:hive2://team-19-jn:5433
```

### 3. Создание тестовой БД
```plaintext
SHOW DATABASES;
CREATE DATABASE test;
SHOW DATABASES;
DESCRIBE DATABASE test;
```

### 4. Выход из Beeline
```bash
Ctrl+C
```

## Загрузка датасета в Hive

### 1. Переходим в корневую директорию на Jump mode

```bash
cd
```

### 2. Скачиваем архив с данными из репозитория

```bash
wget https://github.com/malyushitsky/BigData-mts/raw/refs/heads/main/data/organizations-2000000.zip
```

### 3. Для распаковки архива скачаем unzip, затем распакуем архив

```bash
exit                             # выходим из пользователя hadoop, так как у него нет sudo прав
sudo apt install unzip           # вводим пароль sudo
sudo -i -u hadoop                # возвращаемся обратно в пользователя hadoop
unzip organizations-2000000.zip  # распаковываем архив
```

### 4. Создадим папку на hdfs для хранения датасета перед загрузкой в Hive и поместим туда данные

```bash
hdfs dfs -mkdir /input
hdfs dfs -chmod g+w /input
hdfs dfs -put organizations-2000000.csv /input
```

### 5. Посмотрим на информацию о блоках загруженного файла

```bash
hdfs fsck /input/organizations-2000000.csv
```

### 6. Подключимся к клиенту Beeline

```bash
beeline -u jdbc:hive2://team-19-jn:5433
```

### 7. Сначала создадим в Hive непартиционированную таблицу и загрузим в нее данные

```
use test;
CREATE TABLE IF NOT EXISTS test.dataset_unpartitioned (
    Index INT,
    Organization_Id STRING,
    Name STRING,
    Website STRING,
    Country STRING,
    Description STRING,
    Founded INT,
    Industry STRING,
    Number_of_employees INT)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    TBLPROPERTIES ('skip.header.line.count'='1');

LOAD DATA INPATH '/input/organizations-2000000.csv' INTO TABLE test.dataset_unpartitioned;
```

### 9. Включим динамичесвое партиционирование

```
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
```

### 8. Теперь создадим в Hive партиционированную таблицу и загрузим в нее данные из непартиционированной

```
CREATE TABLE IF NOT EXISTS test.dataset_partitioned (
    Index INT,
    Organization_Id STRING,
    Name STRING,
    Website STRING,
    Country STRING,
    Description STRING,
    Industry STRING,
    Number_of_employees INT)
    PARTITIONED BY (Founded INT)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT INTO TABLE test.dataset_partitioned PARTITION (Founded)
SELECT Index, Organization_Id, Name, Website, Country, Description, Industry, Number_of_employees, Founded
FROM test.dataset_unpartitioned;
```

### 9. Для проверки сдлеаем запрос к партиционированную таблице

```
SELECT COUNT(*) FROM dataset_partitioned;
```

### 10. Также можем посмотреть партиции

```
SHOW PARTITIONS dataset_partitioned;
```

### 11. Для выхода из Beeline

```
!quit
```
