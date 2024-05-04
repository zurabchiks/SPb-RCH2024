Задача:
 - В качестве сервера системы централизованного мониторинга используйте RTR1
 - В качестве системы централизованного мониторинга используйте сборщика prometheus и визуализатор grafana 
 - Система централизованного мониторинга должна быть доступна по адресу http://<IP адрес RTR1>:3000
 - Администратором системы мониторинга должен быть пользователь admin с паролем P@ssw0rd
 - Часовой пояс по умолчанию должен быть Europe/Moscow
 - Разрешите самостоятельную регистрацию новых пользователей
 - Настройте узел системы централизованного мониторинга, в качестве сборщика используйте prometheus node exporter
 - В качестве узлов сети используйте устройства RTR2, SRV1, SRV2
 - Имя узла сети должно соответствовать имени устройства
 - Используйте дашборд, в котором будет видна загрузка ЦП, использование ОП, дисковой подсистемы всех перечисленных устройств. Остальные параметры мониторинга по желанию

>[!NOTE]
>Перед тем как начать установку на RTR1, RTR2, SRV1, SRV2 отключаем SELinux в противном случае он не даст нам настроить службу Node Exporter: 
>```bash
> setenforce 0 && sed -i "s/SELINUX=enforcing/SELINUX=permissive/"  /etc/selinux/config
>```

Далее устанавливаем Prometheus на RTR1. Для установки prometheus выполним команду:

```bash
dnf install golang-github-prometheus
```

Запустим prometheus и добавим его в автозагрузку:

```bash
systemctl enable prometheus –now
```

Проверяем статус службы:

```bash
systemctl status Prometheus
```

>[!NOTE]
>В статусе должно отображаться active(running).

Мониторинг можно производить через веб-интерфейс, введя в адресную строку браузера http://<IP-адрес_сервера>:9090/graph. 

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/61.png)

Далее установим и распакуем Node Exporter:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar -xvzf node_exporter-1.5.0.linux-amd64.tar.gz
```

Затем создадим пользователя nodeusr:

```bash
useradd -rs /bin/false nodeusr
```

Переместим node_exporter в директорию /usr/local/bin/

```bash
mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
```

Создадим службу для запуска:

```bash
nano /etc/systemd/system/node_exporter.service
```

Со следующим содержимым:

```bash
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
```

Запустим node_exporter и добавьте его в автозагрузку:

```bash
systemctl enable node_exporter --now
```

Проверим статус службы:

```bash
systemctl status node_exporter
```

>[!NOTE]
>В статусе должно отображаться active(running).

Для отображения метрик введите в адресную строку браузера http://<IP-адрес_сервера>:9100/metrics.

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/62.png)

По аналогии делаем на Rtr2, Srv1, Srv2

После этого на RTR1 открываем файл конфигурации Prometheus:

```bash
nano /etc/prometheus/prometheus.yml
```

И дополняем секцию scrape_configs следующими строками:

```bash
- job_name: 'rtr2.company.prof'
    static_configs:
      - targets: ['192.168.100.252:9100']

- job_name: 'srv1.company.prof'
    static_configs:
      - targets: ['192.168.100.2:9100']

- job_name: 'srv2.company.prof'
    static_configs:
      - targets: ['192.168.100.3:9100']
```

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/63.png)

Для применения изменений перезапускаем prometheus:

```bash
systemctl restart prometheus
```

Далее установим Grafana на RTR1:

```bash
dnf install grafana
```

Запускаем grafana-server и добавляем его в автозагрузку:

```bash
systemctl enable grafana-server –now
```

Проверяем статус сервера:

```bash
systemctl status grafana-server
```

>[!NOTE]
>В статусе должно отображаться active(running).

Приступим к настройке Grafana, переходим по адресу: http://<IP-адрес>:3000/

Вводим в начале: 
1. Login: admin 
2. Password: admin
3. Затем нам предложит ввести новый пароль, в соответствии с критериями указываем пароль: P@ssw0rd
4. Далее добавим источник данных для Grafana, выбрав Configuration - Data Sources - Add data source.

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/64.png)

Выбираем из списка Prometheus и нажимаем Select.

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/65.png)

>[!NOTE]
>В настройках указываем IP-адрес и порт сервера с Prometheus (http://locallhost:9090), остальные настройки оставляем по умолчанию и нажимаем кнопку Save & Test.

>[!NOTE]
>Далее нам по условию нужно использовать дашборд, в котором будет видна загрузка ЦП, использование ОП, дисковой подсистемы всех перечисленных устройств. 

Для этого импортируем дашборд, выбираем Dashboard - Import. 

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/66.png)

Вводим код для импорта — в нашем случае 1860 (Node Exporter Full):

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/67.png)

Поля можно оставить, как есть. В качестве источника указываем Prometheus и нажимаем Import:

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/68.png)

Результат должен быть следующий:

![screen9](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/69.png)

![screen10](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/70.png)