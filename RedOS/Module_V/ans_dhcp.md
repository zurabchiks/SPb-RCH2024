Задача: 
 - 1. Настройте протокол динамической конфигурации хостов средствами Ansible для группы серверов RTR1
       - В качестве плейбука используйте файл playbook_1.yml в каталоге project_3
       - Плейбук должен содержать действия по настройке протокола динамической конфигурации хостов
           - Адрес сети – согласно топологии
           - Адрес шлюза по умолчанию – адрес маршрутизатора RTR1
           - DNS-суффикс – company.prof
           - Адрес DNS-сервера – адрес маршрутизатора RTR1
           - Адрес NTP-сервера – адрес маршрутизатора RTR1
           - Выдаваемые адреса:
             - Первый адрес – первый адрес сети плюс 5
             - Последний адрес – общее количество адресов в сети разделенное на 2
             - В зависимости от операционной системы, конфигурационные файлы должны располагаться в стандартных каталогах и иметь стандартные имена
             - Использование плагина shell и command НЕ допускается
                 -  Использование запрещенных плагинов обнулит весь пункт при проверке
 
 - 2. Настройте протокол динамической конфигурации хостов средствами Ansible для группы серверов RTR2
       - В качестве плейбука используйте файл playbook_2.yml в каталоге project_3
       - Плейбук должен содержать действия по настройке протокола динамической конфигурации хостов
         - Адрес сети – согласно топологии
         - Адрес шлюза по умолчанию – адрес маршрутизатора RTR2
         - DNS-суффикс – company.prof
         - Адрес DNS-сервера – адрес маршрутизатора RTR2
         - Адрес NTP-сервера – адрес маршрутизатора RTR2
         - Выдаваемые адреса:
             - Первый адрес – общее количество адресов в сети разделенное на 2 плюс 1
             - Последний адрес – последний адрес сети минус 5
             - В зависимости от операционной системы, конфигурационные файлы должны располагаться в стандартных каталогах и иметь стандартные имена
             - Использование плагина shell и command НЕ допускается
                 - Использование запрещенных плагинов обнулит весь пункт при проверке

CLI1

```bash
vim project_3/playbook_1.yml
```

```bash
---
- name: Configuring DHCP on RTR1
  hosts: RTR1
  become: true

# Задаём необходимые переменные
  vars:
    domain_name: company.prof
    plug_subnet: '192.168.15.0'
    plug_subnet_netmask: '255.255.255.0'
    main_subnet: '192.168.100.0'
    main_subnet_netmask: '255.255.255.0'
    first_address_range: '192.168.100.6'
    last_address_range: '192.168.100.127'
    dns_server_address: '192.168.100.254'
    ntp_server_address: '192.168.100.254'
    default_gateway_address: '192.168.100.254'
    interface_work_dhcp: 'enp0s8'

  tasks:
# Устанавливаем пакет dhcp
    - name: Install dhcp
      dnf:
        name: dhcp
        state: present

# Настраиваем DHCP сервер из шаблона
    - name: Setting up DHCP server
      template:
        src: dhcpd.conf.j2
        dest: /etc/dhcp/dhcpd.conf
      notify:
        - Restarted dhcpd

# Указываем на каком интерфейсе должен работать DHCP сервер
    - name: A specific interface to work with
      lineinfile:
        line: 'DHCPARGS={{ interface_work_dhcp }}'
        path: /etc/sysconfig/dhcpd
        state: present
      notify:
        - Restarted dhcpd

# Запускаем и добавляем в автозагрузку DHCP
  handlers:
    - name: Restarted dhcpd
      service:
        name: dhcpd
        state: restarted
        enabled: true
```

Создадим директорию для шаблона

```bash
mkdir project_3/templates
```

Создадим шаблон для конфигурационного файла DHCP сервера

```bash
vim project_3/templates/dhcpd.conf.j2
```

```bash
option domain-name "{{ domain_name }}";

default-lease-time 6000;
max-lease-time 72000;

authoritative;

subnet {{ plug_subnet }} netmask {{ plug_subnet_netmask }} {}

subnet {{ main_subnet }} netmask {{ main_subnet_netmask }}{
  range {{ first_address_range }} {{ last_address_range }};
  option domain-name-servers {{ dns_server_address  }};
  option ntp-servers {{ ntp_server_address }};
  option routers {{ default_gateway_address }};
}
```

Запускаем playbook

```bash
ansible-playbook project_3/playbook_1.yml
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/153.png)

Далее настроим все для RTR2

```bash
vim project_3/playbook_2.yml
```

```bash
---
- name: Configuring DHCP on RTR2
  hosts: RTR2
  become: true

# Задаём необходимые переменные
  vars:
    domain_name: company.prof
    plug_subnet: '192.168.15.0'
    plug_subnet_netmask: '255.255.255.0'
    main_subnet: '192.168.100.0'
    main_subnet_netmask: '255.255.255.0'
    first_address_range: '192.168.100.6'
    last_address_range: '192.168.100.127'
    dns_server_address: '192.168.100.254'
    ntp_server_address: '192.168.100.254'
    default_gateway_address: '192.168.100.254'
    interface_work_dhcp: 'ens34'

  tasks:
# Устанавливаем пакет dhcp
    - name: Install dhcp
      dnf:
        name: dhcp
        state: present

# Настраиваем DHCP сервер из шаблона
    - name: Setting up DHCP server
      template:
        src: dhcpd.conf.j2
        dest: /etc/dhcp/dhcpd.conf
      notify:
        - Restarted dhcpd

# Указываем на каком интерфейсе должен работать DHCP сервер
    - name: A specific interface to work with
      lineinfile:
        line: 'DHCPARGS={{ interface_work_dhcp }}'
        path: /etc/sysconfig/dhcpd
        state: present
      notify:
        - Restarted dhcpd

# Запускаем и добавляем в автозагрузку DHCP
  handlers:
    - name: Restarted dhcpd
      service:
        name: dhcpd
        state: restarted
        enabled: true
```

Запускаем playbook

```bash
ansible-playbook project_3/playbook_2.yml
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/154.png)

После этого переконфигурируем сетевые настройки CLI1 и CLI2 для получения параметров по DHCP и проверяем

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/155.png)


