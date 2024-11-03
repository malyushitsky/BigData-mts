# Гайд по выполнению первого задания:


## Настройка взаимодействия между узлами hadoop


### 1) Подключение к jump node капитана команды
```bash
ssh team@ip # для входа
```

### 2) Предоставить доступ к jump node сокомандникам:
##### 2.1) Собрать внешние ключи сокомандников
##### 2.2) Добавить их в авторизованные ключи на jn
```bash
nano .ssh/authorized_keys # открытие нужного нам файла в редакторе nano
```
- ctrl + v - вставка сохраненных ключей
- ctrl + o - перезапись файла
- ctrl + x - выход из файла

### 3) Создание пользователя hadoop для взаимодействия между всеми узлами кластера
```bash
sudo adduser hadoop # команда для создания нового пользователя
```
- вставить пароль команды для sudo прав 
- придумать пароль для нового пользователя (team19)
- повторить пароль для нового пользователя (team19)
- задать Full Name - hadoop
- остальные поля опциональные, можно прокликать с помощью enter

### 4) Генерация .ssh ключа для нового пользователя hadoop для связи с другими нодами
##### 4.1) Войти в пользователя hadoop
```bash
sudo -i -u hadoop
```
##### 4.2) Генерация ключа
```bash
ssh-keygen
```
##### 4.3) Выводим публичный ключ и копируем его куда-нибудь
```bash
cat .ssh/id_ed25519.pub
```
##### 4.4) Выходим из пользователя hadoop
```bash
exit
```

### 5) Удаление локальной адресации, чтобы хосты знали друг друга по именам
##### 5.1) Редактирование файла hosts на jump node
```bash
sudo nano /etc/hosts # открытие файла
```
- комментируем или удаляем все что находится внутри
- вставляем ip адреса и названия всех узлов кластера и сохраняем файл
- 192.168.1.78    team-19-jn
- 192.168.1.79    team-19-nn
- 192.168.1.80    team-19-dn-0
- 192.168.1.81    team-19-dn-1

### 6) Возвращаемся на jump node 
##### 6.1) Выходим из учетки hadoop
```bash
exit
```
##### 6.2) Переходим на jump node (если были на другой ноде)
```bash
exit
```

### 7) Последовательно заходим на каждый узел кластера и повторяем действия с **4) по 7.2)** (публичные ключи c этапа 5.3) складываем в одном месте)

### 8) Заходим в юзера hadoop и добавляем ключи каждого узла в авторизованные ключи
##### 8.1) Войти в пользователя hadoop
```bash
sudo -i -u hadoop
```
##### 8.2) Открыть файл authorized_keys и добавить туда ключи с сохранением
```bash
nano .ssh/authorized_keys
```

### 9) Передаем созданный файл с ключами на все ноды
```bash
scp .ssh/authorized_keys team-19-nn:/home/hadoop/.ssh/
```
вводим пароль от пользователя hadoop для передачи
```bash
scp .ssh/authorized_keys team-19-dn-0:/home/hadoop/.ssh/
```
вводим пароль от пользователя hadoop для передачи
```bash
scp .ssh/authorized_keys team-19-dn-1:/home/hadoop/.ssh/
```
вводим пароль от пользователя hadoop для передачи

### 10) Запустим скачивание дистрибутива hadoop в сессионном менеджере
```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
```

### 11) Передаем скачанный дистрибутив hadoop на все ноды
```bash
scp hadoop-3.4.0.tar.gz team-19-nn:/home/hadoop
scp hadoop-3.4.0.tar.gz team-19-dn-0:/home/hadoop
scp hadoop-3.4.0.tar.gz team-19-dn-1:/home/hadoop
```

### 12) Переходим на name node и data nodes и распакуем архив с дистрибутивом hadoop
```bash
ssh team-19-nn 
tar -xvzf hadoop-3.4.0.tar.gz 
exit
ssh team-19-dn-0
tar -xvzf hadoop-3.4.0.tar.gz
exit
ssh team-19-dn-1
tar -xvzf hadoop-3.4.0.tar.gz
exit
```

## Настройка самого hadoop


### 13) Переходим на name node для начала настройки кластера
```bash
ssh team-19-nn
```

### 14) Проверяем версию java (должна быть 11)
```bash
java -version
```

### 15) Добавление переменных окружения
##### 15.1) Смотрим где установлена java и сохраняем путь
```bash
readlink -f /usr/bin/java
/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```
##### 15.2) Открываем конфиг файл profile
```bash
nano ~/.profile
```
##### 15.3) Добавляем 3 переменные в файл
```bash
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0 # где развернут дистрибутив
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 # где лежит java
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin # путь для выполнения исполняемых файлов hadoop
```
##### 15.4) Проверка переменных окружения (если ошибок не вылезло, значит компоненты hadoop доступны из домашней директории)
```bash
source ~/.profile
hadoop version
```

### 16) Копируем конфиг profile на data nodes
```bash
scp ~/.profile team-19-dn-0:/home/hadoop
scp ~/.profile team-19-dn-1:/home/hadoop
```

### 17) Переходим в папку дистрибутива
```bash
cd hadoop-3.4.0/etc/hadoop
```

### 18) JAVA_HOME надо добавить в hadoop-env.sh
```bash
nano hadoop-env.sh # открыть файл
```
Вставить в него 
```bash
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```
Сохранить и выйти

### 19) Копируем hadoop-env.sh на data nodes
```bash
scp hadoop-env.sh team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hadoop-env.sh team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

### 20) Настройка файловой системы
##### 20.1) Откроем файл core-site.xml
```bash
nano core-site.xml
```

##### 20.2) Добавим внутрь файла следующие параметры:
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://team-19-nn:9000</value>
    </property>
</configuration>
```

##### 20.3) Откроем файл hdfs-site.xml
```bash
nano hdfs-site.xml
```

##### 20.4) Добавим внутрь файла следующие параметры (фактор репликации 3)
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>
```

##### 20.5) Откроем файл workers
##### 20.6) Меняем localhost на имена нод кластера

- team-19-nn
- team-19-dn-0
- team-19-dn-1

### 21) Копируем все файлы с пункта 20) на data nodes
```bash
scp core-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp core-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
```

## Запуск кластера Hadoop


### 22) Переходим на 2 папки назад
```bash
cd ../../
```

### 23) Форматируем файловую систему
```bash
bin/hdfs namenode -format
```

### 24) Запускаем hadoop
```bash
sbin/start-dfs.sh
```

### 25) Проверяем что все поднялось 
```bash
jps
```

## Настройка nginx для возможности проверки работоспособности нашего кластера через интернет


### 26) Переходим на jump node
```bash
exit
exit
```

### 27) Копируем конфиг для nginx
```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn
```

### 28) Открываем конфиг в nano
```bash
sudo nano /etc/nginx/sites-available/nn
```

### 29) Заменяем внутренности файла на следующие и сохраняем:
```bash
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 9870 default_server;
        #listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                proxy_pass http://team-19-nn:9870;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```
### 30) Сделаем ссылку
```bash
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn
```

### 31) Перезапускаем nginx
```bash
sudo systemctl reload nginx
```

### 32) Откроем в браузере интерфейс hadoop и проверим что все ноды живы
```
176.109.91.21:9870
```






 
