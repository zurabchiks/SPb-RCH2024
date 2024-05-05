Задача:
 - Установка и настройка сервера баз данных
 - В качестве сервера баз данных используйте маршрутизатор RTR2
 - Разверните сервер баз данных на базе MariaDB
 - Пользователь root сервера баз данных должен иметь пароль P@ssw0rd
 - Настройте возможность удаленного подключения к серверу баз данных пользователю root с паролем P@ssw0rd с любых адресов
 - Разверните программное обеспечение для визуального управление базами данных adminer
 - Проверьте корректность входа в adminer пользователя root с паролем P@ssw0rd

Устанавливаем MariaDB:

```bash
dnf install -y mariadb-server
```

Разрешаем автозапуск демона и запускаем его:

```bash
systemctl enable --now mariadb
```

Установим пароль для пользователя root:

```bash
mysqladmin -uroot password
```

>[!NOTE]
>Cистема запросит новый пароль. Указываем пароль как по условию- P@ssw0rd, его нужно ввести дважды.

>[!NOTE]
>Настраиваем удалённое подключение к БД

Правим конфигурационный файл:

```bash
vim /etc/my.cnf.d/mariadb-server.cnf
```

>[!NOTE]
>Необходимо добавить в секцию [ mysqld ] строку: bind-address 0.0.0.0 - тогда сервер будет слушать все интерфейсы компьютера и доступ к БД будет с любых адресов:

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/37.png)

Перезапускаем MariaDB:

```bash
systemctl restart mariadb
```

Проверим:

```bash
ss -tlpn | grep 3306
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/38.png)

Обновим привелегии пользователя root для возможности удалённого подключения:

```bash
mysql -uroot -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'P@ssw0rd' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/39.png)

После проверяем удалённое подключение с RTR2 на RTR1 для пользователя root:

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/40.png)

>[!NOTE]
>Для установки Adminer установим сначала веб-сервер Apache и php. 

Установка веб-сервера Apache:

```bash
dnf install -y httpd httpd-tools
```

Запускаем веб-сервер и добавляем в автозагрузку:

```bash
systemctl enable --now httpd
```

Устанавливаем PHP и часто используемые модули:

```bash
dnf install -y php php-{mysqlnd,dom,simplexml,xml,curl,exif,ftp,gd,iconv,json,mbstring,posix}
```

Добавляем php в автозапуск:

```bash
systemctl enable --now php-fpm
```

Далее создаем директорию:

```bash
mkdir /var/www/html/adminer/
```

Переходим в нее и скачиваем два файла index.php и сам adminer:

```bash
cd /var/www/html/adminer/

wget -O index.php https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-en.php
wget  https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-en.php 
```

Передаем правa на выполнение файла index.php и на директорию /var/www/html/adminer группе apache:

```bash
chown -R apache:apache index.php /var/www/html/adminer/
```

>[!NOTE]
>Изменяем права доступа к каталогу и ко всем файлам внутри /var/www/html/adminer, даем владельцу и группе все права, а другим пользователям только чтение:

```bash
chmod -R 775 /var/www/html/adminer/
```

Перезагружаем apache:

```bash
systemctl restart httpd.service
```

>[!NOTE]
>После этого заходим в базу данных Mariadb. Создаем базу данных adminer, добавляем пользователя adminer с паролем P@ssw0rd и выдаем ему все права:

```bash
create database adminer;
CREATE USER 'adminer'@'localhost' IDENTIFIED BY 'Password';
GRANT ALL ON adminer .* TO 'adminer'@'localhost';
quit;
```

После перезагружаем apache, php, mariadb:

```bash
systemctl restart httpd.service
systemctl restart php-fpm.service
systemctl restart mariadb
```

Проверяем работу adminer, в любом удобном для вас браузере вбиваем:
 <ip-адрес_вашей_машины>/adminer

Появится следующее окно:

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/41.png)

Проверяем корректность входа в adminer пользователя root с паролем P@ssw0rd:

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/42.png)

Нажимаем Login

Все работает!

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/43.png)


