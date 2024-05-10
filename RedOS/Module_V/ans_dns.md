Задача:
 - 1. Настройте перенаправляющий DNS средствами Ansible для группы серверов Router
      - В качестве плейбука используйте файл playbook_1.yml в каталоге project_2
      - Плейбук должен содержать действия по настройке перенаправляющего DNS
         - Используйте bind9
         - Все запросы должны пересылаться внешнему DNS-серверу
             -  Для RTR1 на 77.88.8.8
             -  Для RTR2 на 77.88.8.1
         - В зависимости от операционной системы, конфигурационные файлы и файлы зон должны располагаться в стандартных каталогах и иметь стандартные имена
         - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке
         - Плейбук должен содержать действия по перенастройке адреса DNS-сервера на маршрутизаторах на 127.0.0.1
         - Использование плагина shell и command НЕ допускается
             - Использование запрещенных плагинов обнулит весь пункт при проверке

CLI1

```bash
cd ~/ansible
```

```bash
vim project_2/playbook_1.yml
```

```bash
---
- name: Configuring redirection DNS
  hosts: Router
  become: true 

# определяем переменные
  vars:
    rtr1_external_dns: '77.88.8.8'
    rtr2_external_dns: '77.88.8.1'
    localhost: '127.0.0.1'
    listen_on_port53: any
    allow_query: any
    dnssec_validation: 'no'

  tasks:
# блок для настройки DNS сервера на RTR1 - РедОС
    - name: Configure DNS on REDOS
      block:
        - name: Install bind9 on RedOS
          dnf: 
            name: bind
            state: present

        - name: 'Forward requests to {{ rtr1_external_dns }} on RedOS'
          template:
            src: named.conf.j2
            dest: /etc/named.conf
          notify:
            - Restarted and enabled bind9 on RedOS
      when:
        - ansible_os_family == 'REDOS'

# блок для настройки DNS сервера на RTR2
    - name: Configure DNS 
      block:
        - name: Install bind9
          apt:
            name: bind9
            state: present
            update_cache: true

        - name: 'Forward requests to {{ rtr2_external_dns }} '
          template:
            src: named.conf.options.j2
            dest: /etc/bind/named.conf.options
          notify:
            - Restarted and enabled bind9 
      when:
        - ansible_os_family == 'REDOS'

# переконфигурирование файла resolv.conf на RTR1 и RTR2
    - name: Configure resolv.conf file
      template:
        src: resolv.conf.j2
        dest: /etc/resolv.conf

  handlers:
    - name: Restarted and enabled bind9 on RedOS
      service: 
        name: named
        state: restarted
        enabled: true
```

Создадим директорию для шаблонов

```bash
mkdir project_2/templates
```

Создадим шаблон для конфигурационного файла DNS 

```bash
vim project_2/templates/named.conf.j2
```

```bash
options {
        listen-on port 53 { {{ listen_on_port53 }}; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { {{ allow_query }}; };
        recursion yes;
        dnssec-validation {{ dnssec_validation }};
	forward first;
	forwarders {
		{{ rtr1_external_dns }};
	};
        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Создадим шаблон для конфигурационного файла resolv.conf:

```bash
vim project_2/templates/resolv.conf.j2
```

```bash
search company.prof
nameserver {{ localhost }}
```

```bash
ansible-playbook project_2/playbook_1.yml
```

Проверяем с SRV1 и SRV2

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/152.png)
