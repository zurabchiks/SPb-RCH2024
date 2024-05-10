Задача:
 - 1. В качестве управляемых серверов используйте все устройства, кроме клиентов
     - На устройствах, где нет доступа в Интернет, для установки пакетов используйте установочный диск
     - Доступ разрешен только пользователю sshuser.
     - Доступ пользователю root запрещен в явном виде
     - Доступ по паролю запрещен
 - 2. На CLI1 сгенерируйте пару ключей для устройств с операционной системой RedOS
     - Скопируйте правильный публичный ключ на устройства с операционной системой RedOS.
 - 3. Настройте подключение по SSH по соответствующему ключу.

На всех устройствах за исключением клиентов:

```bash
mkdir /home/sshuser
```

```bash
chow sshuser:sshuser /home/sshuser
```

```bash
cd /home/sshuser
```

```bash
mkdir -p .ssh/
```

```bash
chmod 700 .ssh
```

```bash
touch .ssh/authorized_keys
```

```bash
chmod 600 .ssh/authorized_keys
```

```bash
chown sshuser:sshuser .ssh/authorized_keys
```

```bash
chown sshuser:sshuser .ssh/
```

**CLI1**

Генерация ключевой пары SSH

```bash
ssh-keygen -t rsa ans
```

```bash
mkdir /opt/keys
```

```bash
mv ans* /opt/keys
```

```bash
mkdir .ssh/
```

Создаем config файл для ssh подключений

```bash
nano .ssh/config
```

>[!NOTE]
>На данном скриншоте указано неправильно в графе **IdentityFile** , прописываем следующее:
>```bash
>IdentityFile "/opt/keys/ans"
>```

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/139.png)


```bash
chmod 600 .ssh/config
```

Копируем публичный ключ на устройства:

```bash
ssh-copy-id -i .ssh/ans.pub sshuser@192.168.100.253
```

```bash
ssh-copy-id -i .ssh/ans.pub sshuser@192.168.100.1
```

```bash
ssh-copy-id -i .ssh/astra_ssh_key.pub sshuser@192.168.100.2
```

```bash
ssh-copy-id -i .ssh/ans.pub sshuser@192.168.100.254
```

На всех устройствах за исключением клиентов правим конфигурационный файл **"/etc/ssh/sshd_config"**

```bash
vim /etc/ssh/sshd_config
```

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/140.png)

Перезапускаем сервис

```bash
systemctl restart sshd
```

Проверяем с CLI1

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/141.png)



