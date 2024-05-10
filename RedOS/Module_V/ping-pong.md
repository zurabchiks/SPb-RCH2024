Задача:
 - 1. Настройте узел управления на базе CLI1
      - Установочные файлы находятся в addons_final.iso
      - Вам доступна документация на сайте https://docs.ansible.com/

 - 2.  Сформируйте инвентарь:
      - Создайте файл инвентаря с именем hosts
         - Настройте запуск данного инвентаря по умолчанию
      - Сформируйте группы серверов
         - RTR1 – включается маршрутизатор RTR1
         - RTR2 – включается маршрутизатор RTR2
         - Router – включаются группы серверов RTR1 и RTR2
         - SRV1 – включается сервер SRV1
         - SRV2 – включается сервер SRV2
         - Server – включаются группы серверов SRV1 и SRV2

 - 3.  Реализуйте доступ к серверам с учетом настроек SSH
      - Все параметры должны быть размещены в папке group_vars в качестве переменных
      - Выполните тестовые подключения, добавьте хосты в список известных.

 - 4. Выполните тестовую команду “ping” средствами ansible
      - Убедитесь, что все сервера отвечают “pong” без предупреждающих сообщений
      - Убедитесь, что команды ansible выполняются от пользователя user без использования sudo

 - 5. Создайте необходимую для выполнения задания 2 дня структуру каталогов

**CLI1**

Установим Ansible

```bash
su -
```

```bash
dnf install -y ansible
```

В домашнем каталоге пользователя user создадим директорию для ansible из под пользователя user

```bash
mkdir -p ~/ansible
```

```bash
cd ~/ansible
```

Из под root правим конфигурационный файл "/etc/ansible/ansible.cfg" для указания инвентарного файла в домашней каталоге пользователя user в каталоге ansible

```bash 
vim /etc/ansible/ansible.cfg
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/142.png)

Cоздаем инвентарный файл hosts

```bash
vim ~/ansible/hosts
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/143.png)

Создадим каталог для хранения групповых переменных

```bash
mkdir -p ~/ansible/group_vars
```

В данном каталоге создадим файлы для хранения переменных для групп

```bash
vim ~/ansible/group_vars/all.yml
```

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/144.png)

```bash
vim ~/ansible/group_vars/Server.yml
```

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/145.png)

```bash
vim ~/ansible/group_vars/RTR1.yml
```

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/145.png)

```bash
vim ~/ansible/group_vars/RTR2.yml
```

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/145.png)

Выполним проверку

```bash
cd ~/ansible
```

```bash
ansible -m ping all
```

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/146.png)

Создаём структуру директорий для следующего задания

```bash
mkdir project_{1..5}
```

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/147.png)

