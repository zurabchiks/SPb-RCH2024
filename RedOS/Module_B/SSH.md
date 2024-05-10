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

```bash
nano .ssh/config
```



```bash
chmod 600 .ssh/config
```