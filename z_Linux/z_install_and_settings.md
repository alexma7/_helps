# Содержание

[Установка Debian](#install_debian)  
[Настройка Debian](#settings_debian)  
* [Portainer](#Portainer)  
* [MySql](#MySql)  
* [Mosquitto](#Mosquitto)  
* [Настройка raid](#raid)  


[Настройка Homeassistant](#settings_homeassistant)  
* [Samba](#Samba)  
* [MySql](#MySql)  

<!-- Команды
[](#)  
``` sh

```
-->


# Установка Debian
Скачиваем образ с сайт
Устанавливаем на флешку
Вставляем флешку, выбираем ее для запуска
Открывается окно графической устанвоки дебиан, выбираем все по умолчанию
 * Можно прописать новый айпи
 * При установке ПО, выбираем "install ssh", что бы мы сразу могли подключиться к системе по PUTTY

# Настройка Debian
После установки заходим по root вводим обновление пакетного менеджера и установку sudo
```sh
apt update
apt install sudo -y

sudo -s # можем проверить установку

# Даем доступ пользователю на sudo и делаем релог
usermod -a -G sudo name_user
```

## Востановление сети
``` sh
nmtui 
# edit, autoconnect
# Activate a connection
```


## Настройка raid
``` sh
# Просмотри дисков, запоминаем название нужных
lsblk

# Установим ПО
sudo apt-get install mdadm

# создадии raid-1
sudo mdadm --zero-superblock /dev/sda /dev/sdb
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb

# создадии файловую систему ext4 на рейде
sudo mkfs.ext4 /dev/md0

sudo mkdir /mnt/myraid
sudo mount /dev/md0 /mnt/myraid

```

## Востановление raid
При отключении диска ОС не загружается, на экране будет видно предупреждение, что не может загрузиться такой раздел, введите пароль для востановление  или продолжите.

Надо закоментировать строку из автомонтирования, после этого перегрузиться и сеть будет работать
``` sh

# Ищем устройство, у которого тип связан с рейдом
blkid

# Смотрим все устройства, если у этого устройства нет подраздела, то рейд не построен
lsblk

# Смотрим статистику рейда 
cat /proc/mdstat

# Еще одна проверка рейда
mdadm -D /dev/md*

# Для запуска массива из одного диска, надо остановить рейд
mdadm --stop /dev/md127

# Удалить файл конфигурации 
rm /etc/mdadm/mdadm.conf

# Повторно просканировать диски в поиске массива
mdadm --assemble --scan --verbose

# Прооверяем, рейд должен стать активным
cat /proc/mdstat 

# Добавим пустой диск к рейду
mdadm --add /dev/md0 /dev/sdb

# Можно проверять статус восстановления, идет долго
cat /proc/mdstat 

# Перегружаем систему и наш массив становится с другой цифрой, у меня md127 
reboot







```


## Монтирование дисков
``` sh
# Смотримс список разделов и устройств
sudo fdisk -l

# Ищем наш диск и копирум его айди
sudo blkid
# Открываем файл на редактирование
sudo nano /etc/fstab

# В самом низу пишем коментарий и пишем
UUID=     папка монтирования      файловая система   defaults     0    3
#
#
#
```

## Настройка Samba
``` sh
# создать пользователя
sudo useradd smbuser

# Смена пароля
sudo smbpasswd имя пользователя

# перегрузка
sudo service smbd restart
```


Заменяем конфиг по пути /etc/samba/smb.conf
``` sh
[global]
workgroup = WORKGROUP
security = user
map to guest = bad user
wins support = no
dns proxy = no

[public]
path = /mnt/myraid/public
guest ok = yes
force user = nobody
browsable = yes
writable = yes

[private]
path = /mnt/myraid/private
valid users = @smbgrp
guest ok = no
browsable = yes
writable = yes

[homeassistant]
path = /usr/share/hassio/homeassistant
valid users = @smbgrp
guest ok = no
browsable = yes
writable = yes

```


## Пользователи, группы
``` sh
# Список пользователей, до 1000 - это системные пользователи
cat /etc/passwd

# создаст нового пользователя, useradd -m создаст домашний каталог для пользователя
sudo useradd имя пользователя

# Задать пароль пользователю
sudo passwd имя пользователя

# Категория "скелет", содержит шаблон, для создания пользовательских категорий. Что будет там, то будет при создании у каждого нового пользователя
cd /erc/skel/

# удалить юзера, с флагом -r удалится и каталог пользователя
sudo userdell имя пользователя

# Список групп
groups

# Список групп определенного пользователя
groups пользователь

# Какие пользователи у данной группы
getent group группа

# Создать группу
sudo groupadd имя группы

# Удалить группу
sudo groupadel имя группы

# Добавить пользователя в группу
sudo usermod -a -G имя_группы имя_пользователя

# Удалить пользователя из группы
sudo usermod -R группа пользователь

# Изменить группу 
chgrp имя_группы имя_файла
```

# Настройка Homeassistant
## Samba
В настройка ставим пароль и меняем рабочую группу, потом, в сетевом окружении будет наш сервер с папками.