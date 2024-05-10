Задача:
 - Настройка отказоустойчивой системы централизованного управления авторизацией пользователей
  1. Разверните систему централизованного управления авторизацией пользователей
   - Разверните домен на базе FreeIPA
     - Основной сервер - SRV1
     - Дополнительный сервер (реплика) - SRV2
   - Имя домена - company.prof
   - DNS сервер - интегрированный с IPA
     - Запросы, которые выходят за рамки зоны, пересылаются на виртуальный адрес перенаправляющего DNS-сервера
     - Обратная зона - согласно топологии
     - Все устройства сети должны быть доступны по имени
  - CA сервер - интегрированный с IPA
    - Клиенты домена должны доверять центру сертификации
  - NTP сервер - интегрированный с IPA
    - NTP сервер должен синхронизировать время с отказоустойчивым сервером времени
  - Пароль администратора домена - P@ssw0rd
  2. Настройте систему централизованного управления авторизацией пользователей
     - Создайте пользователей user1, user2 и mon с паролем P@ssw0rd, 
     - Пользователей user1, user2 включите в группу prof
     - Пользователя mon включите в группу admins
     - Создайте правило admin_sudo, разрешающее группе пользователей admins использовать sudo на всех компьютерах в домене без ограничения.
     - Обеспечьте доменному пользователю admin, после успешной авторизации на клиентах, возможность заходить в интерфейс FreeIPA без использования пароля. Для аутентификации и авторизации используйте Kerberos.
     - Используйте Яндекс браузер
  3. Клиентов CLI1 и CLI2 введите в домен FreeIPA 
  - Настройте подключение к системе централизованного мониторинга с использованием FreeIPA
  - Используйте адрес системы централизованного мониторинга http://mon.company.prof:3000
  - Используйте LDAP в качестве аутентификацию по умолчанию
  - В настройках LDAP укажите все имеющиеся серверы
  - Используйте доменного пользователя mon
  - Группа – grafana_admin
  - Тип пользователя – admin

Установим необходимый пакет:

```bash
dnf -y install bind bind-dyndb-ldap ipa-server ipa-server-dns ipa-server-trust-ad
```

Введем команду для интерактивной установки сервера FreeIpa:

```bash
ipa-server-install –mkhomedir
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/92.png)

Проверка запущенных служб и ролей FreeIPA:

```bash
ipactl status
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/93.png)

С клиента проверяем доступ к веб-интерфейса для управления доменом

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/94.png)

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/95.png)

Подготавливаем основной контроллер домена для добавления репликации

В веб-интерфейсе управления необходимо в зоне обратного просмотра создать запись для SRV2

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/96.png)

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/97.png)

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/98.png)

Проверяем

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/99.png)

**SRV2**
В качестве DNS сервера необходимо указать IP адрес SRV1

```bash
vim /etc/resolv.conf
```

![screen9](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/100.png)

Настроить разрешение имен

```bash
vim /etc/hosts
```

![screen10](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/101.png)

Установить инструменты для запуска и настройки

```bash
dnf -y install ipa-client ipa-server-dns
```

>[!NOTE]
>Установить клиента FreeIPA, указав имя домена к которому нужно подключаться (команда ipa-replica-install может сама установить клиента FreeIPA, однако рекомендуется установить клиента используя инструмент ipa-client-install, который автоматически выполняет дополнительные настройки)

```bash
ipa-client-install -d --mkhomedir --enable-dns-updates
```

![screen11](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/102.png)

Проверим

![screen12](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/103.png)

Установить реплику (после запуска команды нужно ввести пароль администратора домена)

```bash
ipa-replica-install
```

Проверяем 

![screen13](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/104.png)

![screen14](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/105.png)

Настраиваем, чтобы запросы которые выходят за рамки зоны, пересылаются на виртуальный адрес перенаправляющего DNS-сервера

![screen15](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/106.png)

Проверяем

![screen16](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/107.png)

Настраиваем DNS записи типа A и PTR для всех устройств

![screen17](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/108.png)

![screen18](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/109.png)

Аналогичным образом добавляем RTR2

Проверяем

![screen19](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/110.png)

![screen20](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/111.png)

![screen21](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/112.png)

Настраиваем динамическое обновление DNS клиентских машин

**SRV1**

Создать ключ для сервера DHCP. Ключ будет автоматически сохранен в файле /etc/bind/rndc.key

```bash
rndc-confgen -a -r /dev/random -b 256
```

В файл конфигурации сервера DNS /etc/bind/named.conf после раздела "options" добавить строчку

```bash
include "/etc/bind/rndc.key";
```

>[!NOTE] 
>Необходимо передать данный файл **"/etc/bind/rndc.key"** на **SRV2** и добавить строку в файл конфигурации сервера DNS /etc/bind/named.conf
>
>Также передать **"/etc/bind/rndc.key"** на **RTR1** и **RTR2** в **/etc**
>
>Запустить WEB-интерфейс управления сервера FreeIPA и в свойствах необходимой прямой зоны в разделе "Политика обновления BIND" добавить запись:
>```bash
>grant rndc-key wildcard * ANY;
>```

![screen22](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/113.png)

В свойствах соответствующей обратной зоны в том же разделе "Политика обновления BIND" добавить запись:

```bash
grant rndc-key wildcard * PTR;
```

![screen23](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/114.png)

Сохраняем и заставляем сервер имен перечитать свою конфигурацию:

```bash
rndc reload
```

перезапускаем службы FreeIPA

```bash
ipactl restart
```

**RTR1**

Добавляем в /etc/dhcp/dhcpd.conf.d/subnets.conf необходимую конфигурацию для работы с DNS на SRV1 и SRV2 c целью динамического обновления записей для клиентов

```bash
vim /etc/dhcp/dhcpd.cond.d/subnets.conf
```

В конечном итоге файл должен выглядеть одинагово как на RTR1 так и на RTR2

```bash
ddns-updates on;
ddns-update-style interim;
include "/etc/rndc.key";
update-static-leases on;

ddns-update-style interim;
update-static-leases on;

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

zone company.prof. {
	primary 192.168.100.2, 192.168.100.3;
	key "rndc-key";
}

zone 100.168.192.in-addr.arpa. {
	primary 192.168.100.2, 192.168.100.3;
	key "rndc-key";
}
```

>[!NOTE]
>файл сгенерированный на SRV1 /etc/bind/rndc.key должен быть передан на DHCP сервера (RTR1, RTR2) в каталог /etc

На **RTR2** конфигурационный файл аналогичен **RTR1**

Перезапускаем DHCP сервера

**RTR1**

```bash
systemctl restart dhcpd
```

**RTR2**

```bash
systemctl restart dhcpd
```

Проверяем через веб-интерфейс freeipa

![screen24](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/115.png)

![screen25](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/116.png)

Проверяем доступ всех устройств по имени

![screen26](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/117.png)

>[!NOTE]
>Для того чтобы клиенты доверяли Центру Сертификации интегрированному с IPA неободимо передать корневой сертификат который расположен на **SRV1** по пути **/etc/ipa/ca.crt**
>
>Передаём данный сертификат на клиентов, а с клиентов выполняем следующие действия
>
>Например

**SRV1**

```bash
scp /etc/ipa/ca.crt cli1@:~/
scp /etc/ipa/ca.crt cli2@:~/
```

**CLI1**

```bash
mv /home/user/ca.crt /etc/pki/ca-trust/source/anchrors
update-ca-trust extract
```

>[!NOTE]
>Проверяем, посредством доступа к веб-интерфейсу freeipa, но уже с корректным сертификатом

![screen27](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/118.png)

**CLI2**

```bash
mv /home/user/ca.crt /etc/pki/ca-trust/source/anchrors
update-ca-trust extract
```

![screen28](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/119.png)

Через веб-интерфейс создаём необходимых пользователей и групп

Создаём группу prof

![screen29](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/120.png)

![screen30](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/121.png)

Создаём пользователя user1

![screen31](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/122.png)

Создаём пользователя user2

![screen32](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/123.png)

Создаём пользователя mon

![screen33](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/124.png)

Добавляем пользователей user1 и user2 в группу prof

![screen34](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/125.png)

![screen35](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/126.png)

Добавляем пользователя mon в группу admins

![screen36](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/127.png)

![screen37](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/128.png)

Создаём правило "admin_sudo"

![screen38](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/129.png)

![screen39](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/130.png)

![screen40](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/131.png)

![screen41](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/131.png)

Или припомощи команд на SRV1 из под пользователя admin

```bash
ipa sudorule-add admin_sudo --cmdcat=all
```

```bash
ipa sudorule-add-user admin_sudo --groups=admins
```

```bash
ipa sudorule-add-host admin_sudo --hosts=ALL
```

![screen42](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/132.png)

Введём CLI1 и CLI2 в домен

**CLI1**

![screen43](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/133.png)

![screen44](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/134.png)

![screen45](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/135.png)

![screen46](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/136.png)

Аналогично настраиваем CLI2

Проверяем

![screen47](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/137.png)

![screen48](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/138.png)