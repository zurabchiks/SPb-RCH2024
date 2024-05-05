Задача:
 - Настройка балансировки и отказоустойчивости DHCP сервера
 - Используйте внутренние сервисы для управления коллективной работой службы DHCPD (failover)
 - Используйте следующие роли для DHCP серверов:
   - RTR1 – primary
   - RTR2 – secondary
 - Используйте следующую конфигурацию взаимодействия DHCP серверов
   - Таймаут, по истечении которого сервер считается не рабочим - 60 секунд
   - Количество запросов, которые могут быть отправлены без обязательного подтверждения – 10
   - Время, в течение которого DHCP сервер будет ждать восстановления канала связи с партнёром – 3600 секунд
   - Индекс разделения работы между DHCP серверами – 128
   - Время, по истечении которого прекращается работа балансировки и сервер отвечает самостоятельно - 3 секунды
 - Переконфигурируйте DHCP сервера для работы failover с учетом отказоустойчивости NAT, DNS, NTP и FreeIPA
   - Имя пира – DHCP
   - Выдаваемые адреса:
   - Первый адрес – первый адрес сети плюс 5
   - Последний адрес – последний адрес сети минус 5
   - Адрес NTP-сервера – адреса доменных NTP серверов  
   - Переконфигурируйте сетевые настройки на CLI1 и CLI2 для получения сетевых параметров по DHCP
   - Проверьте работоспособность DHCP failover на клиентах
 - Клиенты CLI1 и CLI2 должны получать параметры NTP сервера по DHCP
 - Используйте NTP клиент на базе Chrony
 - Используйте часовой пояс Europe/Moscow

Установим dhcp на RTR1:

```bash
dnf install dhcp -y
```

Добавляем в автозапуск:

```bash
systemctl enable --now dhcpd
```

>[!NOTE]
>В файле "/etc/dhcp/dhcpd.conf" - будем хранить только настройки отказоустойчивости;
Поскольку всё кроме настройки отказоустойчивости на RTR1 и RTR2 будет одинаковым - вынесем конфиги в отдельную дирректорию:

```bash
mkdir /etc/dhcp/dhcpd.conf.d
cat /etc/dhcp/dhcpd.conf | tee -a /etc/dhcp/dhcpd.cond.d/subnets.conf
```

>[!NOTE]
>Для того чтобы основной файл /etc/dhcp/dhcpd.conf - ссылался на то что лежит в дирректории /etc/dhcp/dhcpd.conf.d/, сотрём в нём всё и пропишим опции include

```bash
echo include \"/etc/dhcp/dhcpd.conf.d/subnets.conf\"\; > /etc/dhcp/dhcpd.conf
```

Теперь прописываем опции касающиеся отказоустойчивости в "/etc/dhcp/dhcpd.conf" выше опций include:

```bash
vim /etc/dhcp/dhcpd.conf
```

```bash
failover peer "DHCP" {
        primary;                          # Основной сервер
        address 192.168.100.253;          # Адрес текущего сервера, который будет использоваться для обмена информацией между DHCP - серверами
        port 647;                         # Порт на котором будет слушать этот сервер
        peer address 192.168.100.252;     # Адрес другого сервера (secondary)
        peer port 847;                    # Порт другого сервера (secondary)
        max-response-delay 60;            # Сколько секунд прождать, чтобы посчитать второй сервер недоступным
        max-unacked-updates 10;           # Сколько максимум пакетов bind-update может отправить второй сервер
        mclt 3600;                        # Максимальное время на которое одному серверу разрешено время lease, не сказав об этом второму серверу (только на primary указывается)
        split 128;                        # Распределение адресов 50/50% (только на primary указывается)
        load balance max seconds 3;       # Если пришёл DHCPDISCOVER или DHCOREQUEST и в течение 3-х секунд второй сервер не ответил, хотя очередь была его, то балансировка игнорируется и клиенту отвечает этот сервер
}

include "/etc/dhcp/dhcpd.conf.d/subnets.conf";
```

>[!NOTE]
>Необходимо указать failover peer для subnet-ов, в файле "/etc/dhcp/dhcpd.conf.d/subnets.conf" и параметры согласно заданию:

```bash
vim /etc/dhcp/dhcpd.conf.d/subnets.conf
```

```bash
option domain-name "company.prof";
option domain-name-servers 192.168.100.2, 192.168.100.3;

default-lease-time 6000;
max-lease-time 72000;

authoritative;

subnet 192.168.15.0 netmask 255.255.255.0 {}

subnet 192.168.100.0 netmask 255.255.255.0{
  option ntp-servers 192.168.100.2, 192.168.100.3;
  option routers 192.168.100.254;
  pool {
        failover peer "DHCP";
        range 192.168.100.6 192.168.100.249;
  }
}
```

Перезагружаем DHCP сервер

```bash
systemctl restart dhcpd
```

Проверяем:

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/83.png)

RTR2 
Аналогичным образом настраиваем RTR2 только в качестве SECONDARY

```bash
dnf install dhcp -y
systemctl enable --now dhcpd
mkdir /etc/dhcp/dhcpd.conf.d
cat /etc/dhcp/dhcpd.conf | tee -a /etc/dhcp/dhcpd.cond.d/subnets.conf
echo include \"/etc/dhcp/dhcpd.conf.d/subnets.conf\"\; > /etc/dhcp/dhcpd.conf
```

>[!NOTE]
>Редактируем параметры в /etc/dhcp/dhcpd.conf на DHCP secondary - для настройки этого сервера как secondary

```bash
vim /etc/dhcp/dhcpd.conf

failover peer "DHCP" {
	secondary;
	address 192.168.100.252;
	port 847;
	peer address 192.168.100.253;
	peer port 647;
	max-response-delay 60;
	max-unacked-updates 10;
	load balance max seconds 3;
}

include "/etc/dhcp/dhcpd.conf.d/subnets.conf";
```

>[!NOTE]
>Файл "/etc/dhcp/dhcpd.conf.d/subnets.conf" должен быть точно таким же, как и на RTR1

```bash
vim /etc/dhcp/dhcpd.conf.d/subnets.conf

option domain-name "company.prof";
option domain-name-servers 192.168.100.2, 192.168.100.3;
 
default-lease-time 6000;
max-lease-time 72000;

authoritative;

subnet 192.168.15.0 netmask 255.255.255.0 {}

subnet 192.168.100.0 netmask 255.255.255.0{
  option ntp-servers 192.168.100.2, 192.168.100.3;
  option routers 192.168.100.254;
  pool {
	failover peer "DHCP";
	range 192.168.100.6 192.168.100.249;
  }
}
```

Перезапускаем DHCP сервер

```bash
systemctl restart dhcpd
```

Проверяем функциональность работы DHCP failover, включаем CLI1 и CLI2 и получаем настройки сетевых параметров по DHCP и проверяем журналы
RTR1 - на запрос получения IP адреса 192.168.100.247 передал его на peer, тоесть на RTR2

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/84.png)

>[!NOTE]
>RTR2 - выдал IP адрес 192.168.100.247, а запрос на получения IP адреса 192.168.100.128 передал его на peer, тоесть на RTR1

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/85.png)

Теперь приступим к настройке NTP-сервера. Установим chrony на SRV1 и на SRV2 и добавим его в автозагрузку:

```bash
dnf install chrony -y
systemctl enable --now chronyd.service
```

>[!NOTE]
>Файл конфигурации chrony располагается по пути /etc/chrony.conf, добавим в конец файла разрешающее правило для синхронизации времени в локальной сети 192.168.100.0/24

```bash
vim /etc/chrony.conf 
```

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/86.png)

Сохраняем файл и перезапускаем службу:

```bash
systemctl restart chronyd
```

По аналогии настраиваем SRV2

После чего на CLI1 и CLI2 устанавливаем chrony и добавляем его в автозапуск:

```bash
dnf install chrony
systemctl enable --now chronyd.service
```

Переходим в файл конфигурации и меняем сервера в конфиге на два настроенных нами сервера. Результат должен быть следующим:

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/87.png)

Сохраняем файл и перезапускаем службу:

```bash
systemctl restart chronyd
```

По аналогии настраиваем CLI2

Проверяем результат:

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/88.png)

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/89.png)

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/90.png)

![screen9](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/91.png)

