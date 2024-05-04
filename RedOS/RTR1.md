1. Настройте возможность отправки и получения ICMP запросов между RTR1 и RTR2 по внешним адресам:

   - Отправка и получение ICMP запросов на ISP должна быть запрещена

Настроим GRE туннель между RTR1 и RTR2.

Открываем nmtui выбираем изменить подключение>добавить>ip-туннель. 

Задаем имя профиля и устройство. 

Режим работы выбираем GRE. 

Родительский интерфейс выбираем тот, который у нас смотрит в сторону ISP (enp0s3).

Задаем Локальный IP (ip на интерфейсе RTR1 в сторону ISP) 

Задаем Удаленный IP (ip на интерфейсе RTR2 в сторону ISP)

Переходим к Конфигурации IPV4

Задаем IPV4 адрес для туннеля

Результат должен быть следующий:

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/8.png)

После этого нажимаем OK и активируем интерфейс

На RTR2 делаем аналогичные настройки, результат должен быть таким:

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/9.png)

>[!NOTE] 
>Для корректной работы протокола динамической маршрутизации требуется увеличить параметр TTL на интерфейс туннеля:

```bash
nmcli connection modify tun1 ip-tunnel.ttl 64 
```

Проверяем: 

```bash
ip -c a
```

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/10.png)

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/11.png)

Попробуем пропинговать с RTR1 и с RTR2:

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/12.png)

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/13.png)

Проверим получится ли через ISP пропинговать ip адреса наших gre-туннелей:

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/14.png)

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/15.png)

