Задача:
 - Настройка отказоустойчивости сервера времени
 - Группа серверов VRRP включает в себя маршрутизаторы RTR1 и RTR2
 - Создайте группу серверов Keepalived со следующими параметрами:
 - Имя группы – NTP
 - Иерархия группы – RTR1 -> RTR2
 - Идентификатор группы – 123
 - Приоритет - 110 и 100 соответственно
 - Виртуальный адрес группы - последний адрес сети
 - Интервал рассылки сообщений - 1 секунда
 - Время, после которого сервер с более высоким приоритетом заберет обратно себе роль мастера – 30 секунд
 - Обеспечьте автозапуск конфигурации

Добавляем в конфигурационный файл keepalived.conf группу для NTP
RTR1

```bash
vim /etc/keepalived/keepalived.conf
```

```bash
...
vrrp_instance NTP {
    state MASTER
    interface enp0s8
    virtual_router_id 123
    priority 110
    advert_int 1
    preempt_delay 30
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
...
vrrp_instance NTP {
    state BACKUP
    interface enp0s8
    virtual_router_id 123
    priority 100
    advert_int 1
    preempt_delay 30
    virtual_ipaddress {
	192.168.100.254/24
    }
}
```

На устройствах RTR1 и RTR2 перезапускаем службу keepalived

```bash
systemctl restart keepalived
```
