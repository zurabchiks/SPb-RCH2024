1. Базовая настройка устройств

   - Настройте имена устройств согласно топологии
   
     ![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/1.png)
   
   - Используйте полное доменное имя, кроме ISP
   - Используйте строчные буквы, кроме ISP
   - Настройте адресацию устройств согласно топологии
   - Адрес сети – согласно топологии
     - Для RTR1 – последний адрес сети минус 1
     - Для RTR2 – последний адрес сети минус 2
     - Для SRV1 – первый адрес сети плюс 1
     - Для SRV2 – первый адрес сети плюс 2
     - Для CLI1 – десятый адрес сети
     - Для CLI2 – двадцатый адрес сети

   - Адрес шлюза по умолчанию:
     - Для SRV1 – адрес маршрутизатора RTR1
     - Для SRV2 – адрес маршрутизатора RTR2
     - Для CLI1 – адрес маршрутизатора RTR1
     - Для CLI2 – адрес маршрутизатора RTR2
     - Для SW – адрес маршрутизатора RTR1 c метрикой 50 и адрес маршрутизатора RTR2 с метрикой 100

   - DNS-суффикс – company.prof
     - Используйте в качестве домена поиска
     - Адрес DNS-сервера:
     - Для RTR1 – адрес 77.88.8.8
     - Для RTR2 – адрес 77.88.8.1
     - Для SRV1 – адрес маршрутизатора RTR1
     - Для SRV2 – адрес маршрутизатора RTR2
     - Для CLI1 – адрес маршрутизатора RTR1
     - Для CLI2 – адрес маршрутизатора RTR2

   - На всех устройствах, кроме CLI1 и CLI2, создайте пользователя sshuser с паролем P@ssw0rd
     - Пользователь sshuser должен иметь возможность запуска утилиты sudo без дополнительной аутентификации(без ввода даже своего пароля).

Настраиваем интерфейсы через nmtui. Для начала настроим ISP. Первый интерфейс выбираем сетевой мост, два остальных локальная сеть (RTR1_ISP, RTR2_ISP).  Далее настраиваем интерфейсы enp0s8, enp0s9

 ![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/2.png)

 ![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/3.png)

Далее настраиваем интерфейсы до ISP (enp0s3) и до локальной сети (enp0s8) на RTR1 и RTR2. Настроим RTR1:

 ![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/4.png)

 ![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/5.png) 

Далее настраиваем аналогично RTR2 в соответствии с таблицей маршрутизации:

 ![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/6.png)

 ![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/7.png)

Создадим пользователя без домашнего каталога:

``` bash
useradd -M sshuser
passwd sshuser
```

Разрешим данному пользователю использовать sudo без доп.аутентификации:

``` bash
echo “sshuser ALL=(ALL) NOPASSWD: ALL” >> etc/sudoers
```