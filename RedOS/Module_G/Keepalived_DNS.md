Задача:
 - Группа серверов VRRP включает в себя маршрутизаторы RTR1 и RTR2
 - Создайте группу серверов Keepalived со следующими параметрами:
 - Имя группы – DNS
 - Иерархия группы - RTR1 -> RTR2
 - Идентификатор группы – 53
 - Приоритет - 110 и 100 соответственно
 - Виртуальный адрес группы - последний адрес сети
 - Интервал рассылки сообщений - 1 секунда
 - Время, после которого сервер с более высоким приоритетом заберет обратно себе роль мастера – 30 секунд
 - Обеспечьте автозапуск конфигурации
 - Внесите изменения в настройки перенаправляющих DNS с учетом работы keepalived

>[!NOTE]
>Добавляем в конфигурационный файл keepalived.conf группу для перенаправляющего DNS

RTR1

```bash
vim /etc/keepalived/keepalived.conf
```

```bash
…
vrrp_instance DNS {
    state MASTER
    interface enp0s8
    virtual_router_id 53
    priority 110
    preempt_delay 30
    advert_int 1
    virtual_ipaddress {
	192.168.100.254/24
    }
}
```

RTR2

```bash
vim /etc/keepalived/keepalived.conf
```

```bash
…
vrrp_instance DNS {
    state BACKUP
    interface enp0s8
    virtual_router_id 53
    priority 100
    advert_int 1
    preempt_delay 30
    virtual_ipaddress {
	192.168.100.254/24
    }
}
```

На устройствах RTR1 и RTR2 перезапускаем службу keepalived:

```bash
systemctl restart keepalived
```

Далее на RTR1 и на RTR2 устанавливаем DNS и добавляем его в автозапуск:

```bash
dnf install bind -y
systemctl enable named –now
```

>[!NOTE]
>После чего на RTR1 и RTR2 в конфигурационных файлах перенаправляющего DNS сервера необходимо указать, что теперь мы слушаем запросы только с localhost и только с виртуального адреса - 192.168.100.254

```bash
vim /etc/named.conf
```

RTR1

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/78.png)

RTR2 

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/79.png)

После этого перезагружаем службу:

```bash
systemctl restart named
```

Заходим после этого на SRV1 и на SRV2 и через nmtui меняем адрес DNS-сервера

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/80.png)

По аналогии настраиваем SRV2

Проверяем: 

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/81.png)

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/82.png)