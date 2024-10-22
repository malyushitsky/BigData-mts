# Гайд по выполнению первого задания:


## Настройка взаимодействия между узлами hadoop


### 1) Подключение к jump node капитана команды
ssh team@ip для входа

### 2) Предоставить доступ к jump node сокомандникам:
##### 2.1) Собрать внешние ключи сокомандников
##### 2.2) Добавить их в авторизованные ключи на jn
nano .ssh/authorized_keys - открытие нужного нам файла в редакторе nano
ctrl + v - вставка сохраненных ключей
ctrl + o - перезапись файла
ctrl + x - выход из файла

### 3) Запустим скачивание дистрибутива hadoop в сессионном менеджере
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz

### 4) Создание пользователя hadoop для взаимодействия между всеми узлами кластера
sudo adduser hadoop - команда для создания нового пользователя
вставить пароль команды для sudo прав 
придумать пароль для нового пользователя (team19)
повторить пароль для нового пользователя (team19)
задать Full Name - hadoop
остальные поля опциональные, можно прокликать с помощью enter

### 5) Генерация .ssh ключа для нового пользователя hadoop для связи с другими нодами
##### 5.1) Войти в пользователя hadoop
sudo -i -u hadoop
##### 5.2) Генерация ключа
ssh-keygen
##### 5.3) Выводим публичный ключ и копируем его куда-нибудь
cat .ssh/id_ed25519.pub

### 6) Удаление локальной адресации, чтобы хосты знали друг друга по именам
##### 6.1) Редактирование файла hosts на jump node
sudo nano /etc/hosts - открытие файла
комментируем или удаляем все что находится внутри
вставляем ip адреса и названия всех узлов кластера и сохраняем файл
192.168.1.78    team-19-jn
192.168.1.79    team-19-nn
192.168.1.80    team-19-dn-0
192.168.1.81    team-19-dn-1

### 7) Возвращаемся на jump node 
##### 7.1) Выходим из учетки hadoop
exit
##### 7.2) Переходим на jump node
exit 

### 8) Последовательно заходим на каждый узел кластера и повторяем действия с **4) по 7.2)** (публичные ключи c этапа 5.3) складываем в одном месте)

### 9) Заходим в юзера hadoop и добавляем ключи каждого узла в авторизованные ключи
##### 9.1) Войти в пользователя hadoop
sudo -i -u hadoop
##### 9.2) Открыть файл authorized_keys и добавить туда ключи с сохранением 
nano .ssh/authorized_keys

### 10) Передаем созданный файл с ключами на все ноды
scp .ssh/authorized_keys team-19-nn:/home/hadoop/.ssh/
вводим пароль от пользователя hadoop для передачи
scp .ssh/authorized_keys team-19-dn-0:/home/hadoop/.ssh/
вводим пароль от пользователя hadoop для передачи
scp .ssh/authorized_keys team-19-dn-1:/home/hadoop/.ssh/
вводим пароль от пользователя hadoop для передачи

### 11) Передаем скачанный дистрибутив hadoop на все ноды
scp hadoop-3.4.0.tar.gz team-19-nn:/home/hadoop
scp hadoop-3.4.0.tar.gz team-19-dn-0:/home/hadoop
scp hadoop-3.4.0.tar.gz team-19-dn-0:/home/hadoop

### 12) Переходим на name node и data nodes и распакуем архив с дистрибутивом hadoop
ssh team-19-nn 
tar -xvzf hadoop-3.4.0.tar.gz 
exit
ssh team-19-dn-0
tar -xvzf hadoop-3.4.0.tar.gz
exit
ssh team-19-dn-1
tar -xvzf hadoop-3.4.0.tar.gz
exit


## Настройка самого hadoop


### 13) Переходим на name node для начала настройки кластера
ssh team-19-nn

### 14) Проверяем версию java (должна быть 11)
java -version

### 15) Добавление переменных окружения
##### 15.1) Смотрим где установлена java и сохраняем путь
readlink -f /usr/bin/java
/usr/lib/jvm/java-11-openjdk-amd64/bin/java
##### 15.2) Открываем конфиг файл profile
nano ~/.profile
##### 15.3) Добавляем 3 переменные в файл
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0 - где развернут дистрибутив
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 - где лежит java
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin - путь для выполнения исполняемых файлов hadoop
##### 15.4) Проверка переменных окружения (если ошибок не вылезло, значит компоненты hadoop доступны из домашней директории)
source ~/.profile
hadoop version

### 16) Копируем конфиг profile на data nodes
scp ~/.profile team-19-dn-0:/home/hadoop
scp ~/.profile team-19-dn-1:/home/hadoop

### 17) Переходим в папку дистрибутива
cd hadoop-3.4.0/etc/hadoop

### 18) JAVA_HOME надо добавить в hadoop-env.sh
nano hadoop-env.sh - открыть файл
Вставить в него JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
Сохранить и выйти

### 19) Копируем hadoop-env.sh на data nodes
scp hadoop-env.sh team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hadoop-env.sh team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop

### 20) Настройка файловой системы
##### 20.1) Откроем файл core-site.xml
nano core-site.xml
##### 20.2) Добавим внутрь файла следующие параметры:
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://team-19-nn:9000</value>
    </property>
</configuration>

##### 20.3) Откроем файл hdfs-site.xml
##### 20.4) Добавим внутрь файла следующие параметры (фактор репликации 3)
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
</configuration>

##### 20.5) Откроем файл workers
##### 20.6) Меняем localhost на имена нод кластера
team-19-nn
team-19-dn-0
team-19-dn-1

### 21) Копируем все файлы с пункта 20) на data nodes
scp core-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp core-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp hdfs-site.xml team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-19-dn-0:/home/hadoop/hadoop-3.4.0/etc/hadoop
scp workers team-19-dn-1:/home/hadoop/hadoop-3.4.0/etc/hadoop


## Запуск кластера Hadoop


### 22) Переходим на 2 папки назад
cd ../../

### 23) Форматируем файловую систему
bin/hdfs namenode -format

### 24) Запускаем hadoop
sbin/start-dfs.sh

### 25) Проверяем что все поднялось 
jps


## Настройка nginx для возможности проверки работоспособности нашего кластера через интернет


### 26) Переходим на jump node
exit
exit

### 27) Копируем конфиг для nginx
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn

### 28) Открываем конфиг в nano
sudo nano /etc/nginx/sites-available/nn

### 29) Заменяем внутренности файла на следующие и сохраняем:

\#\#
\# You should look at the following URL's in order to grasp a solid understanding
\# of Nginx configuration files in order to fully unleash the power of Nginx.
\# https://www.nginx.com/resources/wiki/start/
\# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
\# https://wiki.debian.org/Nginx/DirectoryStructure
\#
\# In most cases, administrators will remove this file from sites-enabled/ and
\# leave it as reference inside of sites-available where it will continue to be
\# updated by the nginx packaging team.
\#
\# This file will automatically load configuration files provided by other
\# applications, such as Drupal or Wordpress. These applications will be made
\# available underneath a path with that package name, such as /drupal8.
\#
\# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
\#\#

\# Default server configuration
\#
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


\# Virtual Host configuration for example.com
\#
\# You can move that to a different file under sites-available/ and symlink that
\# to sites-enabled/ to enable it.
\#
\#server {
\#       listen 80;
\#       listen [::]:80;
\#
\#       server_name example.com;
\#
\#       root /var/www/example.com;
\#       index index.html;
\#
\#       location / {
\#               try_files $uri $uri/ =404;
\#       }
\#}

### 30) Сделаем ссылку
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn

### 31) Перезапускаем nginx
sudo systemctl reload nginx

### 32) Откроем в браузере интерфейс hadoop и проверим что все ноды живы
176.109.91.21:9870