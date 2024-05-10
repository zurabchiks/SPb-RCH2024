Задача: 
 - 1. Настройте сервер времени средствами Ansible для группы серверов Router
       - В качестве плейбука используйте файл playbook_1.yml в каталоге project_4
       - Плейбук должен содержать действия по установке и настройке сервера времени
          - Используйте сервер времени на базе Chrony
          - Используйте часовой пояс Europe/Moscow
          - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке

CLI1

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

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/156.png)

Проверим с RTR1 и RTR2 клиентов NTP

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/157.png)

