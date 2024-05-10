Задача: 
 - 1. Настройте NFS сервер средствами Ansible для группы маршрутизаторов Router
      - В качестве плейбука используйте файл playbook_1.yml в каталоге project_5
      - Плейбук должен содержать действия по настройке NFS сервера
         - Настройте общий доступ к директории /opt/data

 - 2. Настройте NFS клиента средствами Ansible для группы серверов Server
      - В качестве плейбука используйте файл playbook_2.yml в каталоге project_5
      - Плейбук должен содержать действия по настройке NFS клиента
         - На SRV1 настройте автоматическое подключение NFS каталога
             - Используйте локальную точку монтирования /mnt/data
             - Используйте общую папку на RTR1
         - На SRV2 настройте автоматическое подключение NFS каталога
             - Используйте локальную точку монтирования /mnt/data
             - Используйте общую папку на RTR2

CLI1

```bash
vim project_5/playbook_1.yml
```

```bash
---
- name: Configuring NFS server
  hosts: Router
  become: true 

# Определяем необзодимые переменные
  vars:
    share_folder: /opt/data
    access_rights: '0777'
    access_network_share: '192.168.100.0/24'

  tasks:
# Назначаем права на директорию которая будет использоваться для общего доступа
    - name: 'Setting full read and write access on {{ share_folder }}'
      file:
        path: '{{ share_folder }}'
        state: directory
        mode: '{{ access_rights }}'

# Настраиваем NFS сервер для RTR1
    - name: Configuring NFS server on RTR1
      block: 
        - name: Install NFS packakes on RTR1
          dnf:
            name: 
              - nfs-utils
              - nfs4-acl-tools
            state: present

        - name: Configuring exports file on RTR1
          lineinfile:
            line: '{{ share_folder }} {{ access_network_share }}(rw,insecure,nohide,all_squash,anonuid=1000,anongid=1000,no_subtree_check)'
            state: present
            path: /etc/exports
          notify:
            - Restarted nfs-server
      when:
        - ansible_os_family == 'REDOS'

# Настраиваем NFS сервер для RTR2
    - name: Configuring NFS server on RTR2
      block:
        - name: Install NFS server on RTR2
          apt:
            name: nfs-kernel-server
            state: present
            update_cache: true

        - name: Configuring exports file on RTR2
          lineinfile:
            line: '{{ share_folder }} {{ access_network_share }}(rw,nohide,all_squash,anonuid=1000,anongid=1000,no_subtree_check)'
            state: present
            path: /etc/exports
          notify:
            - Restarted nfs-kernel-server
      when:
        - ansible_os_family == 'REDOS'

# Опционально, публичный вариант данного задания не запрещает использования в данном playbook модуль command
    - name: Re-export the share
      command: 
        cmd: exportfs -rav

# Перезапускаем и добавляем в автозагрузку NFS сервера
  handlers:
    - name: Restarted nfs-server
      service:
        name: nfs-server
        state: restarted
        enabled: true

    - name: Restarted nfs-kernel-server 
      service:
        name: nfs-kernel-server
        state: restarted
        enabled: true
```

Запускаем playbook

```bash
ansible-playbook project_5/playbook_1.yml
```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/158.png)

Настраиваем NFS клиента

CLI1

```bash
vim project_5/playbook_2.yml
```

```bash
---
- name: Configuring NFS client on group Server
  hosts: Server
  become: true

# Задаём необзодимые переменные
  vars:
    mount_point: /mnt/data
    access_rights_mount_point: '0777'
    nfs_share_folder_path: /opt/data
    srv1_nfs_server: '192.168.100.254'
    srv2_nfs_server: '192.168.100.253'

  tasks:
# Устанавливаем пакет nfs-common
    - name: Install client nfs
      apt:
        name: nfs-common
        state: present
        update_cache: true

# Создаём точку монтирования
    - name: 'Create mount point {{ mount_point }}'
      file:
        path: '{{ mount_point }}'
        state: directory
        mode: '{{ access_rights_mount_point }}'

# Добавляем запись в fstab, чтобы SRV1 монтировал общий ресурс с RTR1
    - name: Automatic mounting SRV1
      lineinfile:
        path: /etc/fstab
        line: '{{ srv1_nfs_server }}:{{ nfs_share_folder_path }} {{ mount_point }} nfs rw,sync,hard,intr 0 0'
        state: present
      when:
        - ansible_hostname == 'srv1'

# Добавляем запись в fstab, чтобы SRV2 монтировал общий ресурс с RTR2
    - name: Automatic mounting SRV2
      lineinfile:
        path: /etc/fstab
        line: '{{ srv2_nfs_server }}:{{ nfs_share_folder_path }} {{ mount_point }} nfs rw,sync,hard,intr 0 0'
        state: present
      when:
        - ansible_hostname == 'srv2'

# Перезагружаем для применения автомонтирования
    - name: Reboot
      reboot:
```

Запускаем playbook

```bash
ansible-playbook project_5/playbook_2.yml
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/159.png)

Проверяем с SRV1 и SRV2

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/160.png)



