Задача: 
 - 1. Настройте динамическую трансляцию адресов средствами Ansible для группы серверов RTR1
      - В качестве плейбука используйте файл playbook_1.yml в каталоге project_1
      - Плейбук должен содержать действия по настройке динамической трансляцию адресов
         - Используйте firewalld
         - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке
            
 - 2. Настройте динамическую трансляцию адресов средствами Ansible для группы серверов RTR2
      - В качестве плейбука используйте файл playbook_2.yml в каталоге project_1
      - Плейбук должен содержать действия по настройке динамической трансляцию адресов
         - Используйте iptables
         - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке

CLI1    

Создадим playbook_1.yml

```bash
cd ~/ansible
```

```bash
vi project_1/playbook_1.yml
```

```bash
---
- name: Setting up dynamic address translation by Firewalld
  hosts: RTR1
  become: true

# определяем переменные
  vars:
    external_int: 'ens33'
    internal_int: 'ens34'
    external_zone: 'dmz'
    internal_zone: 'trusted'

  tasks:
# включаем перессылку пакетов (forwarding)
    - name: Set ip forwarding on in /proc and in the sysctl file and reload necessary
      sysctl:
        name: net.ipv4.ip_forward
        value: '1'
        sysctl_set: true
        state: present
        reload: true

# устанавливаем пакет firewalld
    - name: Install Firewalld
      dnf:
        name: firewalld
        state: present

# включаем и добавляем в автозагрузку firewalld
    - name: Started and enabled Firewalld
      service:
        name: firewalld
        state: started
        enabled: true

# распределяем интерфейсы по зонам: внешний интерфейс добавляется в зону dmz, внутренний интерфейс добавляется в зону trusted
    - name: Distribution of interfaces to the corresponding zones
      firewalld:
        zone: '{{ item.zone }}'
        interface: '{{ item.interface }}'
        permanent: true
        state: enabled
        immediate: yes
      loop:
        - { zone: '{{ external_zone }}', interface: '{{ external_int }}' }
        - { zone: '{{ internal_zone }}', interface: '{{ internal_int }}' }

# включаем маскировку трафика на зону dmz в которой находится внешний интерфейс
    - name: 'Turn on masquerade on {{ external_zone }}'
      firewalld:
        zone: '{{ external_zone }}'
        masquerade: 'yes'
        permanent: true
        state: enabled
        immediate: true
```

```bash
ansible-playbook project_1/playbook_1.yml
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/148.png)

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/149.png)

Создадим playbook_2.yml

```bash
cd ~/ansible
```

```bash
vim project_1/playbook_2.yml
```

```bash
---
- name: Setting up dynamic address translation by Iptables
  hosts: RTR2
  become: true

# определяем переменные
  vars:
    external_int: 'eth0'
    lan_network: '192.168.100.0/24'
    iptables_save_path: /etc/iptables.rules

  tasks:
# включаем перессылку пакетов (forwarding)
    - name: Set ip forwarding on in /proc and in the sysctl file and reload necessary
      sysctl:
        name: net.ipv4.ip_forward
        value: '1'
        sysctl_set: true
        state: present
        reload: true

# Устанавливаем Iptalbes
    - name: Install Iptables
      apt:
        name: iptables
        state: present
        update_cache: yes

# Включаем маскировку трафика из локальной сети через внешний интерфейс
    -name: Turn on masquerade
      iptables:
        table: nat
        chain: POSTROUTING
        jump: MASQUERADE
        out_interface: '{{ external_int }}'
        protocol: all
        source: '{{ lan_network }}'
        destination: '0.0.0.0/0'

# Сохраняем правила Iptables в файл
    - name: Save current state of the iptables in system file
      community.general.iptables_state:
        state: saved
        path: '{{ iptables_save_path }}'

# Добавляем выгрузку правил из ранее сохранённого файла во время загрузки системы
    - name: Enable auto-loading of rules iptables
      lineinfile:
        path: /etc/network/interfaces
        line: 'pre-up iptables-restore < {{ iptables_save_path }}'
        state: present
```

Настраиваем доступ в Интернет для установки необходимой коллекции

```bash
echo nameserver 77.88.8.8 >> /etc/resolv.conf
exit
```

Из подпользователя user ставим недостоющую коллекцию

```bash
ansible-galaxy collection install community.general
```

Запускаем playbook_2

```bash
ansible-playbook project_1/playbook_2.yml
```

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/150.png)

Проверяем с SRV2

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/151.png)