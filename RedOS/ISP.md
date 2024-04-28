1. Настройте адресацию на интерфейсах:

   - Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP
   - Интерфейс, к которому подключен RTR1, имеет адрес 11.11.11.1/24
   - Интерфейс, к которому подключен RTR2, имеет адрес 22.22.22.1/24

2. Настройте DHCP сервер:

   - Пул INET1
     - Адрес сети – 11.11.11.0/24
     - Выдаваемые адреса – 11.11.11.100 – 11.11.11.200
     - Адрес шлюза по умолчанию – 11.11.11.1
     - Адрес DNS-сервера – 11.11.11.1

   - Пул INET2
     - Адрес сети – 22.22.22.0/24
     - Выдаваемые адреса – 22.22.22.100 – 22.22.22.200
     - Адрес шлюза по умолчанию – 22.22.22.1
     - Адрес DNS-сервера – 22.22.22.1

3. Настройте перенаправляющий DNS сервер:
   - Все запросы должны пересылаться на внешний DNS сервер по адресу 8.8.8.8
   - DNS сервер должен быть доступен по адресу 11.11.11.1 для RTR1
   - DNS сервер должен быть доступен по адресу 22.22.22.1 для RTR2

4. Настройте динамическую трансляцию адресов для сети 11.11.11.0/24 и 22.22.22.0/24 
   - RTR1 и RTR2 должны иметь выход в Интернет


Устанавливаем dhcp-server:

``` bash
dnf install dhcp
```

Далее переходим к настройке dhcp пула:

``` bash
nano /etc/dhcp/dhcpd.conf
```

Пишем туда следующие параметры:

``` bash
#INET1
subnet 11.11.11.0 netmask 255.255.255.0 {
    range 11.11.11.100 11.11.11.200;
    option routers 11.11.11.1;
    option domain-name-servers 11.11.11.1;
}

#INET2
subnet 22.22.22.0 netmask 255.255.255.0 {
    range 22.22.22.100 22.22.22.200;
    option routers 22.22.22.1;
    option domain-name-servers 22.22.22.1;
}
```

После чего сохраняем изменения, перезагружаем службу и добавляем ее в автозапуск:

``` bash
systemctl enable --now dhcpd
```
Установка BIND:

```bash
dnf install bind -y
```

Разрешаем автозапуск:

```bash
systemctl enable named –now
```

Проверяем корректность работы:

```bash
systemctl status named
```

Открываем на редактирование конфигурационный файл bind:

```bash
nano /etc/named.conf
```

И редактируем следующее в блоке options:

```bash
listen-on port 53 { 127.0.0.1; 11.11.11.1; 22.22.22.1 };
listen-on-v6 port 53 { none; };
allow-query { any; };
forward first;
forwarders { 8.8.8.8; };
```

Устанавливаем на ISP,RTR1,RTR2 nftables:

```bash 
dnf install -y nftables
```

Создаем файл и указываем в нем настройки:

```bash
nano /etc/nftables/isp.nft
```

```bash
table ip nat {
        chain postrouting {
                type nat hook postrouting priority filter; policy accept;
                ip saddr 11.11.11.0/24 oifname "enp0s3" counter packets 0 bytes 0 masquerade
                ip saddr 22.22.22.0/24 oifname "enp0s3" counter packets 0 bytes 0 masquerade
        }
}
```

Добавляем в основной файл ссылку на данный файл:

```bash
nano /etc/sysconfig/nftables.conf
```

Прописываем строку:

```bash
include “/etc/nftables/isp.nft”
```

После этого запускаем и добавляем в автозагрузку nftables:

```bash
systemctl enable --now nftables
```

На устройствах RTR1 и RTR2 производим аналогичную настройку nftables для доступа SRV1 и SRV2 к сети Интернет.