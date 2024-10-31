
# Гайд по выполнению первого задания:

### 1) Подключение
```bash
ssh team@host
```

### 2) Переходим на пользователя hadoop
```bash
sudo -i -u hadoop
```

### 3) Переходим на name node
```bash
ssh team-19-nn
```

### 4) Переходим в папку дистрибутива
```bash
cd hadoop-3.4.0/etc/hadoop
```

 
### 5) Настройка конфигов MapReduce и YARN

##### 5.1) Заходим в конфиг MapReduce
```bash
nano mapred-site.xml
```

##### 5.2) Вставляем в конфиг MapReduce
```bash
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.application.classpath</name>
                <value>$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
        </property>
</configuration>
```

##### 5.3) Заходим в конфиг YARN
```bash
nano yarn-site.xml
```

##### 5.4) Вставляем в конфиг YARN 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <name>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <name>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```

### 6) Копируем конфиги на data nodes
```bash
scp mapred-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp mapred-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop

scp yarn-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp yarn-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

### 7) Запуск YARN
```bash
cd ../../
sbin/start-yarn.sh
```

### 8) Запуск historyserver
```bash
mapred --daemon start historyserver
```

### 9) Конфиги для веб-интерфейсов YARN и historyserver

##### 9.1) Переходим на jn
```bash
ssh team-19-jn
```

##### 9.2) Выходим из учетки hadoop
```bash
exit
```

##### 9.3) Выполняем
```bash
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/ya
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/dh
```

##### 9.4) YARN. Открываем конфиг и заменяем порт в listen и proxy_pass на порт 8088
```bash
 sudo nano /etc/nginx/sites-available/ya
```

##### 9.5) historyserver. Открываем конфиг и заменяем порт в listen и proxy_pass на порт 19888
```bash
 sudo nano /etc/nginx/sites-available/dh
```

### 10) Включаем хосты
```bash
 sudo ln -s /etc/nginx/sites-available/ya /etc/nginx/sites-enabled/ya
 sudo ln -s /etc/nginx/sites-available/dh /etc/nginx/sites-enabled/dh
```

### 11) Запуск nginx
```bash
 sudo systemctl reload nginx
 exit
```
 
### 12) Проверка работы UI

##### 12.1) YARN. Переходим в браузере по адресу
```bash
host:8088
```

##### 12.2) historyserver. Переходим в браузере по адресу
```bash
host:19888
```

### 13) Остановка сервисов

##### 13.1) Выполняем
```bash
ssh team@host
sudo -i -u hadoop
ssh team-19-nn
cd hadoop-3.4.0/
```

##### 13.2) Останавливаем historyserver
```bash
mapred --daemon stop historyserver
```

##### 13.3) Останавливаем YARN (могут быть оповещения о невозможности остановки и будет сделан kill)
```bash
sbin/stop-yarn.sh
```

##### 13.4) Останавливаем dfs
```bash
sbin/stop-dfs.sh
```

##### 13.5) Проверка nn
```bash
jps
```

##### 13.6) Проверка dn-0
```bash
ssh team-19-dn-0
jps
exit
```

##### 13.7) Проверка dn-1
```bash
ssh team-19-dn-1
jps
exit
```

### 14) Настройка веб-интерфейсов NodeManager

##### 14.1) Подключаемся
```bash
ssh team@host
sudo -i -u hadoop
```

##### 14.2) Переключаемся на dn-0 и открываем yarn-site.xml
```bash
ssh team-19-dn-0
nano /home/hadoop/hadoop-3.4.0/etc/hadoop/yarn-site.xml
```

##### 14.3) Вставляем в конфиг 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
       <property>
      		<name>yarn.nodemanager.webapp.address</name>
      		<value>0.0.0.0:8042</value>
  	</property>
       <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```

##### 14.4) Переключаемся на dn-1 и открываем yarn-site.xml
```bash
ssh team-19-dn-1
nano /home/hadoop/hadoop-3.4.0/etc/hadoop/yarn-site.xml
```

##### 14.5) Вставляем в конфиг 
```bash
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->

       <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
       <property>
      		<name>yarn.nodemanager.webapp.address</name>
      		<value>0.0.0.0:8042</value>
  	</property>
       <property>
                <name>yarn.nodemanager.env-whitelist</name>
                <value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```

##### 14.6) Переходим на jn
```bash
ssh team-19-jn
```

##### 14.7) Выходим из учетки hadoop
```bash
exit
```

##### 14.7) Копируем конфигурации nginx для каждого интерфейса NodeManager
```bash
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/nm-0
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/nm-1
```

##### 14.8) Открываем файл и заменяем порт в listen на 8043, порт в proxy_pass на 8042
```bash
sudo nano /etc/nginx/sites-available/nm-0
```

##### 14.9) Открываем файл и заменяем порт в listen на 8044, порт в proxy_pass на 8042
```bash
sudo nano /etc/nginx/sites-available/nm-1
```

##### 14.10) Включаем NodeManager в nginx
```bash
sudo ln -s /etc/nginx/sites-available/nm-0 /etc/nginx/sites-enabled/nm-0
sudo ln -s /etc/nginx/sites-available/nm-1 /etc/nginx/sites-enabled/nm-1
```

##### 14.11) Перезапускаем nginx
```bash
sudo systemctl reload nginx
```

##### 14.12) Переходим в браузере по адресам и проверяем
```bash
host:8043
host:8044
```
