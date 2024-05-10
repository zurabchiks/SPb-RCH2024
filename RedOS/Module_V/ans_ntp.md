Задача: 
 - 1. Настройте сервер времени средствами Ansible для группы серверов Router
       - В качестве плейбука используйте файл playbook_1.yml в каталоге project_4
       - Плейбук должен содержать действия по установке и настройке сервера времени
          - Используйте сервер времени на базе Chrony
          - Используйте часовой пояс Europe/Moscow
          - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке

 - 2. Настройте NTP клиента средствами Ansible для группы серверов Server
   -  В качестве плейбука используйте файл playbook_2.yml в каталоге project_4
   - Проект должен содержать действия по установке и настройке NTP клиента
      - Используйте NTP клиент на базе Chrony
      - Используйте часовой пояс Europe/Moscow
      - Использование плагина shell и command НЕ допускается
         - Использование запрещенных плагинов обнулит весь пункт при проверке

CLI1

```bash
vim project_4/playbook_1.yml
```

```bash
---
- name: Configuring NTP on Router group
  hosts: Router
  become: true

# Задаём необходимые переменные
  vars:
    time_zone: 'Europe/Moscow'
    external_ntp_server_rtr1: '1.ru.pool.ntp.org'
    external_ntp_server_rtr2: '2.ru.pool.ntp.org'
    allowed_network: '192.168.100.0/24'

  tasks:
# Настраиваем часовой пояс
    - name: 'Setting the time zone {{ time_zone }}'
      timezone:
        name: '{{ time_zone }}'

# Устанавливаем пакет chrony
    - name: Install chrony
      package:
        name: chrony
        state: present

# Настраиваем NTP сервер из шаблона на RTR1
    - name: Customization chrony.conf file on RTR1
      template:
        src: rtr1_chrony.conf.j2
        dest: /etc/chrony.conf
      notify:
        - Restarted chronyd
      when:
        - ansible_os_family == 'REDOS'

# Настраиваем NTP сервер из шаблона на RTR2
    - name: Customization chrony.conf file on RTR2
      template:
        src: rtr2_chrony.conf.j2
        dest: /etc/chrony/chrony.conf
      notify:
        - Restarted chronyd
      when:
        - ansible_os_family == 'REDOS'

# Запускаем и добавляем в автозагрузку chronyd
  handlers:
    - name: Restarted chronyd
      service:
        name: chronyd
        state: restarted
        enabled: true
```

Создадим директорию для шаблонов

```bash
mkdir project_4/templates
```

Создадим шаблон конфигурационного файла NTP сервера для RTR1

```bash
vim project_4/templates/rtr1_chrony.conf.j2
```

```bash
server {{ external_ntp_server_rtr1 }}

driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
allow {{ allowed_network }}

keyfile /etc/chrony.keys
ntsdumpdir /var/lib/chrony
leapsectz right/UTC
logdir /var/log/chrony
```

Создадим шаблон конфигурационного файла NTP сервера для RTR2

```bash
vim project_4/templates/rtr2_chrony.conf.j2
```

```bash
pool {{ external_ntp_server_rtr2 }} iburst
allow {{ allowed_network }}

keyfile /etc/chrony/chrony.keys
driftfile /var/lib/chrony/chrony.drift
logdir /var/log/chrony
maxupdateskew 100.0
rtcsync
makestep 1 3
```

Запустим playbook

```bash
ansible-playbook project_4/playbook_1.yml
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/156.png)

Приступаем к настройке клиента

```bash
vim project_4/playbook_2.yml
```

```bash
---
- name: Configuring NTP clients
  hosts: Server
  become: true

# Задаём необходимые переменные
  vars:
    time_zone: 'Europe/Moscow'
    ntp_srv1_sync: '192.168.100.254'
    ntp_srv2_sync: '192.186.100.253'

  tasks:
# Настраиваем часовой пояс
    - name: 'Setting the time zone {{ time_zone }}'
      timezone:
        name: '{{ time_zone }}'

    - name: Install chrony
      package:
        name: chrony
        state: present

# Настраиваем NTP клиент на SRV1
    - name: Customization chrony.conf on SRV1
      lineinfile:
        regexp: '^pool'
        line: 'pool {{ ntp_srv1_sync }} iburst prefer'
        path: /etc/chrony/chrony.conf
      notify:
        - Restarted chronyd
      when:
        - ansible_hostname == 'srv1'

# Настраиваем NTP клиент на SRV2
    - name: Customization chrony.conf on SRV2
      lineinfile:
        regexp: '^pool'
        line: 'pool {{ ntp_srv2_sync }} iburst prefer'
        path: /etc/chrony/chrony.conf
      notify:
        - Restarted chronyd
      when:
        - ansible_hostname == 'srv2'

# Запускаем и добавляем в автозагрузку chronyd
  handlers:
    - name: Restarted chronyd
      service:
        name: chronyd
        state: restarted
        enabled: true
```

Запускаем playbook

```bash
ansible-playbook project_4/playbook_2.yml
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/157.png)

Проверим с RTR1 и RTR2 клиентов NTP

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/158.png)

