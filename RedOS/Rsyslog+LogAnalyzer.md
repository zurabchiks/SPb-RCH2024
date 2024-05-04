Задача:
 - В качестве сервера системы централизованного журналирования используйте RTR2
 - В качестве системы централизованного журналирования используйте Rsyslog совместно с веб панелью LogAnalyzer
 - Настройте Rsyslog
 - Настройте взаимосвязь сервера баз данных с Rsyslog
 - В качестве сервера баз данных используйте MariaDB на RTR2
 - Имя базы данных: Syslog
 - Пользователь базы данных: rsyslog
 - Пароль пользователя базы данных: rsyslogpwd
 - В базу данных должны записываться только сообщения об ошибках и более важные
 - Настройте возможность приема сообщений по протоколам TCP и UDP по порту 514
 - Установите LogAnalyzer
 - В качестве веб-сервера используйте Apache
 - Файлы LogAnalyzer должны располагаться в папке /var/www/html/loganalyzer
 - Используйте базу данных, с которой работает Rsyslog
 - Веб панель LogAnalyzer должна быть доступна по адресу http://<IP адрес RTR2>/loganalyzer
 - Для авторизации в веб панели LogAnalyzer необходимо использовать пользователя admin с паролем P@ssw0rd
 - Требовать, чтобы пользователь вошел в систему
 - Настройте централизованный сбор журналов с хостов RTR1, SRV1, SRV2
 - Уровень журналирования – сообщения об ошибках и более важные
 - RTR1 использует протокол UDP
 - SRV1 и SRV2 использует протокол TCP


Для начала, необходимо установить пакеты для работы rsyslog с базой данных MariaDB:

```bash
dnf install -y rsyslog rsyslog-mysql
```

Затем импортировать схему базы данных rsyslog в MariaDB файл по пути "/usr/share/doc/rsyslog/mysql-createDB.sql":

```bash
mysql -uroot -p < /usr/share/doc/rsyslog/mysql-createDB.sql
```

Также необходимо создать пользователя rsyslog и предоставьте ему права к базе данных Syslog:

```bash
mysql -uroot -p
GRANT ALL PRIVILEGES ON Syslog.* TO 'rsyslog'@'localhost' IDENTIFIED BY 'rsyslogpwd';
FLUSH PRIVILEGES;
exit;
```

Далее настраиваем rsyslog:

>[!NOTE]
>Для разрешения серверу принимать соединение по TCP и UDP в конфигурационном файле /etc/rsyslog.conf раскомментируйте строки в секции Modules:

```bash
vim /etc/rsyslog.conf 
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/44.png)

Далее создаем файл mysql.conf для работы MariaDB в качестве сервера баз данных для rsyslog:

```bash
vim /etc/rsyslog.d/mysql.conf
```

И указываем следующее:

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/45.png)

>[!NOTE]
>По условию от нас требуют, чтобы в базу данных записывались только сообщения об ошибках и более важные. Мы указываем модуль с которым должен работать демон rsyslog (load=”ommysql”) и указываем чтобы в базу данных Syslog с пользователем rsyslog и паролем rsyslogpwd вносились только сообщения об ошибках (*.err) и более важные сообщения- критические (*.crit).

Проверяем конфигурационные файлы:

```bash
rsyslogd -N1
```

Если все правильно результат будет следующий:

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/46.png)

Для применения внесенных изменений перезагрузите службу rsyslog:

```bash
systemctl restart rsyslog
```

И проверяем статус:

```bash
systemctl status rsyslog.service
```

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/47.png)

Переходим в каталог /tmp:

```bash
cd /tmp
```

Скачиваем архив LogAnalyzer и распаковываем его следующими командами:

```bash
wget http://download.adiscon.com/loganalyzer/loganalyzer-4.1.13.tar.gz
tar xzf loganalyzer-4.1.13.tar.gz
```

Далее перемещаем приложение в каталог веб-сервера по умолчанию:

```bash
mv loganalyzer-4.1.13/src /var/www/html/loganalyzer
```

>[!NOTE]
>Создаем файл конфигурации config.php в каталоге loganalyzer и предоставляем локальному пользователю (user) соответствующие права для внесения изменений в файл:

```bash
cd /var/www/html/loganalyzer
touch config.php
chown user:user config.php
chmod 777 config.php
```

>[!NOTE]
> - Открываем браузер и в адресной строке прописываем http://<ip_адрес_вашей_машины/loganalyzer/. 
> - Будет открыта страница с ошибкой, т. к. приложение еще не настроено. В тексте описания ошибки нажимаем на слово «here» для продолжения настройки.

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/48.png)

Шаг №1. На стартовой странице нажмите Next.

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/49.png)

Шаг №2. На втором шаге производится проверка на наличие файла конфигурации config.php. В него будут записаны последующие настройки. 

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/50.png)

>[!NOTE]
>Рядом с обнаруженным файлом обязательно должна отображаться надпись «Writeable». При ее отсутствии в файл не могут быть записаны настройки.

Шаг №3 На третьей странице задается базовая конфигурация приложения. В ней мы указываем базу данных Syslog, пользователя rsyslog и пароль rsyslogpwd, в указанных местах нажимаем конпку yes, затем нажимаем кнопку next.

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/51.png)

Шаг №4. Если настройка проведена корректно, будет открыта страница с предупреждением о внесении изменений в конфигурационный файл config.php. Подключение к базе данных успешно установлено. Нажмите Next.

![screen9](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/52.png)

Шаг №5. На данной странице будет выведено сообщение об успешном создании таблицы в базе данных, а также отобразится количество успешных операций и ошибок.

![screen10](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/53.png)

Шаг №6. Здесь будет предложено создать первого пользователя для веб-приложения. Данный пользователь будет иметь административные права и доступ к центру администрирования. Создаем пользователя admin с паролем P@ssw0rd.

![screen11](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/54.png)

Шаг №7. На этой странице необходимо снова указать источник, который ведет системный журнал.

>[!NOTE]
>!ВАЖНО! Нужно исправить в графе Database Tablename на SystemEvents, в противном случае наши таблицы не загрузит и выдаст ошибку.

![screen12](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/55.png)

Шаг №8. Если настройка выполнена верно, будет открыта страница с сообщением об успешной установке LogAnalyzer. Нажмите Finish для перехода на страницу аутентификации.

![screen13](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/56.png)

После успешной аутентификации будут открыты логи в удобном для просмотра формате.

Вводим логин (admin) и пароль (P@ssw0rd) 

![screen14](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/57.png)

Все работает!

![screen15](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/58.png)

>[!NOTE]
>Для создания правила на клиенте воспользуйтесь директорией rsyslog.d, которая является дополнительной для конфигурационных файлов

Для rtr1 создаем два файла erors.conf и crit.conf и указываем в них следующие параметры:

![screen16](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/59.png)

![screen17](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/60.png)

Сохраняем эти два файла и перезагружаем службу:

```bash
systemctl reload rsyslog 
```

По аналогии создаем два файла на SRV1 и на SRV2.