Задача: 
 - Группа серверов VRRP включает в себя маршрутизаторы RTR1 и RTR2
 - Создайте группу серверов Keepalived со следующими параметрами:
 - Имя группы – NAT
 - Иерархия группы - RTR1 -> RTR2
 - Идентификатор группы – 69
 - Приоритет - 110 и 100 соответственно
 - Виртуальный адрес группы - последний адрес сети
 - Интервал рассылки сообщений - 1 секунда
 - Время, после которого сервер с более высоким приоритетом заберет обратно себе роль мастера – 30 секунд
 - Обеспечьте автозапуск конфигурации

Установим пакет keepalived на RTR1 и RTR2:

```bash
dnf install -y keepalived
```

Описываем конфигурационный файл keepalived.conf для RTR1:

```bash
vim /etc/keepalived/keepalived.conf

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance NAT {
    state MASTER
    interface enp0s8
    virtual_router_id 69
    priority 110
    advert_int 1
    preempt_delay 30
    virtual_ipaddress {
	192.168.100.254/24
    }
}
```

>[!NOTE]
>где: 
> - global_defs: Этот блок используется для задания глобальных настроек Keepalived.
> - router_id LVS_DEVEL: Этот параметр устанавливает идентификатор маршрутизатора (Router ID) для Keepalived. Идентификатор маршрутизатора используется для уникальной идентификации маршрутизатора в группе. В данном случае, идентификатор маршрутизатора установлен как "LVS_DEVEL".
> - vrrp_instance NAT: Это блок, который определяет настройки для конкретной группы VRRP. В данной конфигурации создается группа VRRP с именем "NAT".
> - state MASTER: Указывает, что этот маршрутизатор (или Keepalived инстанс) является активным мастером (первичным).
> - interface enp0s8: Здесь указывается интерфейс, к которому привязана эта группа VRRP.
> - virtual_router_id 10: Устанавливает уникальный идентификатор группы VRRP. Этот идентификатор должен быть одинаковым на всех маршрутизаторах, участвующих в группе.
> - priority 110: Устанавливает приоритет этого маршрутизатора в группе VRRP. Маршрутизатор с более высоким приоритетом станет мастером (активным) при старте Keepalived или после восстановления.
> - advert_int 1: Устанавливает интервал отправки объявлений VRRP в секундах. В данном случае, объявления будут отправляться каждую секунду.
> - preempt_delay 30: Время, после которого сервер с более высоким приоритетом заберет обратно себе роль мастера
> - virtual_ipaddress: Здесь указывается виртуальный IP-адрес, который будет назначен группе VRRP. Этот IP-адрес будет переноситься между мастером и резервными маршрутизаторами в случае сбоя.

Похожие настройки делаем на RTR2:

```bash
vim /etc/keepalived/keepalived.conf

global_defs {
   router_id LVS_DEVEL
}

vrrp_instance NAT {
    state BACKUP
    interface enp0s8
    virtual_router_id 69
    priority 100
    advert_int 1
    preempt_delay 30
    virtual_ipaddress {
	192.168.100.254/24
    }
}
```

Запускаем на RTR1 и RTR2 keepalived:

```bash
systemctl enable --now keepalived
```

Проверяем статус:

```bash
systemctl status keepalived
```

RTR1

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/71.png)

RTR2

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/72.png)

>[!NOTE]
>Теперь в случае, если RTR1 перестанет работать, RTR2 получит адрес, который мы указали в настройках и вместо роли BACKUP, получит роль MASTER до того момента пока RTR1 не начнет свою работу.

>[!NOTE]
>Переконфигурируем сетевые настройки SRV1 и SRV2 c учетом настроек отказоустойчивости динамической трансляции адресов

SRV1:
 1. Открываем nmtui
 2. Изменяем шлюз на адрес 192.168.100.254

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/73.png)

По аналогии меняем шлюз на SRV2

Проверяем:

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/74.png)

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/75.png)

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/76.png)

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/77.png)


