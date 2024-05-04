Задача:
 -  На RTR1 настройте RAID массив
 -	Уровень дискового массива RAID 1.
 -	Используйте имя дискового массива md0.
 -  Используйте два неразмеченных жестких диска.
 -  Используйте 100% дискового пространства
 -  Используйте файловую систему ext4.
 -	Настройте автоматическое монтирование дискового массива.
 -	Точка монтирования /opt/data.

Для создания RAID массива используем диски sdb и sdc.

На новых неразмеченных дисках необходимо создать разделы при помощи утилиты cfdisk:

```bash
cfdisk /dev/sdb
```

Откроется псевдографическая утилита, на данном этапе необходимо указать метку раздела — значение по умолчанию, GPT.

![screen1](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/15.png)

Далее необходимо выбрать Новый для создания нового раздела.

![screen2](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/16.png)

>[!NOTE] 
>Далее указывается размер создаваемого раздела, по умолчанию указывается все свободное пространство целевого диска. Нажатием Enter происходит соглашение с размером раздела.
 
После чего необходимо перейти в опцию меню Тип, чтобы указать, какой формат разметки выбрать. По умолчанию выбирается Linux Filesystem, но для создания RAID-массива потребуется отформатировать диск в Linux RAID.

![screen3](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/17.png)

В указанном списке выбирается Linux RAID. 

![screen4](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/18.png)

Запишем изменения.

![screen5](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/19.png)

Вручную пишем слово yes, чтобы принять изменения и отформатировать диск.

![screen6](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/20.png)

После внесенных правок необходимо покинуть утилиту cfdisk, выбрав параметр Выход.

![screen7](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/21.png)

>[!NOTE]
>Необходимо повторить данную процедуру также и со вторым добавленным диском.

Проверим:

```bash
lsblk
```

![screen8](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/22.png)

Далее необходимо собрать RAID1-массив:

```bash
mdadm --create /dev/md0 –-level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

![screen9](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/23.png)

>[!NOTE]
>где:
 > mdadm — обращение к утилите mdadm;
 > --create — создать новый массив;
 > /dev/md0 — указывается, как будет называться логический том   RAID-массива, /dev/mdX, где Х — порядковый номер RAID-массива, а mdX — зарезервированное имя устройства для RAID в Linux;
 > --level=1 — указывается уровень RAID-массива. 1 — указывает на RAID1 («Зеркало»);
 > --raid-devices=2 — указывается количество устройств, добавленных в RAID-массив;
 > /dev/sdb1 /dev/sdc1 — перечисляются диски, которые добавим в RAID-массив.

Проверим:

```bash
lsblk
```

![screen10](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/24.png)

После того, как массив собран, его необходимо сохранить:

```bash
mdadm --detail --scan --verbose | tee -a /etc/mdadm.conf
```

>[!NOTE]
>После того, как массив собран, необходимо форматировать его в необходимую файловую систему, например ext4 и настроить автоматическое монтирование при загрузке в директорию /opt/data.

Форматируем в ext4:

```bash
mkfs.ext4 /dev/md0
```

![screen11](https://github.com/zurabchiks/SPb-RCH2024/blob/main/RedOS/Pic/25.png)

Далее необходимо создать точку монтирования данного массива:

```bash
mkdir /opt/data
```

После этого необходимо добавить запись в файл /etc/fstab:

```bash
vim /etc/fstab
```






