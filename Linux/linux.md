# Содержание

[Программы](#Программы)  
* [Portainer](#Portainer)  
* [MySql](#MySql)  
* [Mosquitto](#Mosquitto)  
* [iotlink](#iotlink)  
* [](#)  
* [](#)  

[Файловая система](#Файловая_система)  





<!-- Команды
[](#)  
``` bash

```
-->


## Portainer
## Установка портейнера
Вводим
```sh
sudo armbian-config
```

Выбираем в следующей последовательности:
```sh
Software System and 3rd party software install
жмем OK
Softy 3rd party applications installer
жмем OK
Отмечаем мышью
Hassio Home assistant smarthome suite
жмем Install
Ждем, после загрузки видим, что добавлен Docker, нажимаем Install и установка завершена.
Далее можно заходить в браузер и заходить по адресу http://192.168.1.**:8123
```
## Обновление портейнера

``` bash
#  скачаем новый портейнер
docker pull portainer/portainer-ce

# остановим и удалим старый
docker stop portainer
docker rm portainer

# Создаем новый контейнер и запускаем его
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

```

## MySql
MySql - по умолчанию отключен доступ для подключения с удаленного компа. Для доступа надо закоментить строку
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
Комментируем строчку #"bind-address = 127.0.0.1"

## Mosquitto <a name="Mosquitto"></a>
``` sh
sudo apt-get update
sudo apt-get upgrade

sudo wget http://repo.mosquitto.org/debian/mosquitto-repo.gpg.key
sudo apt-key add mosquitto-repo.gpg.key
cd /etc/apt/sources.list.d/
sudo wget http://repo.mosquitto.org/debian/mosquitto-jessie.list
sudo apt-get update

sudo apt-get install mosquitto

sudo apt-get install mosquitto mosquitto-clients

sudo /etc/init.d/mosquitto stop - остановка москито

sudo nano /etc/mosquitto/mosquitto.conf
Заносим туда следующее
# Place your local configuration in /etc/mosquitto/conf.d/
#
# A full description of the configuration file is at
# /usr/share/doc/mosquitto/examples/mosquitto.conf.example

pid_file /var/run/mosquitto.pid

persistence true
persistence_location /var/lib/mosquitto/

log_dest topic

log_type error
log_type warning
log_type notice
log_type information

connection_messages true
log_timestamp true

include_dir /etc/mosquitto/conf.d`

sudo /etc/init.d/mosquitto start - старт сервера

Проверка установки, открываем 2 окно SSH
mosquitto_sub -d -t hello/world - подписываемся на паблик
mosquitto_pub -d -t hello/world -m "Hello from Terminal window 2!" - с другого отправляем

sudo /etc/init.d/mosquitto status - проверка статуса сервиса
```


# iotlink

настройка 
скачаиваем и устанавливаем https://iotlink.gitlab.io/downloads.html
Открываем файл
```
C:\ProgramData\IOTLink\Configs\configuration.yaml
```
вводим
```
    username: user
    password: pass
    ...
    
    hostname: 192.168.1.1
```

Настроим конфиг монитора в файле:
    
```
    C:\ProgramData\IOTLink\Addons\WindowsMonitor\config.yaml
```
Установим проверку дисков на 1800 секунд, статусы вкл/выкл и тд на 10 сек, проверка датчиков на 1 минуту
```
  # CPU:
  #   enabled: true
  #   interval: 10
  #   cacheable: true
  # Memory:
  #   enabled: true
  #   interval: 10
  #   cacheable: true
  HardDrive:
    enabled: true
    interval: 1800
    cacheable: true
  Power:
    enabled: true
    interval: 10
    cacheable: true
  # NetworkInfo:
  #   enabled: true
  #   interval: 10
  #   cacheable: true
  #   SystemInfo:
  #     enabled: true
  #     interval: 60
  #     cacheable: true
  # IdleTime:
  #   enabled: true
  #   interval: 10
  #   cacheable: true
  #   inSeconds: true
  # Uptime:
  #   enabled: true
  #   interval: 60
  #   cacheable: true
  # Display-Info:
  #   enabled: true
  #   interval: 60
  #   cacheable: true
  # Display-Screenshot:
  #   enabled: true
  #   interval: 60
  #   cacheable: false
 ```


## Файловая система
ls - вывод списка файлов
  * ключ -l  - дополнительная информация
  * ключ -a  - показать скрытые данные

в линуксе нет корневых каталогов для каждого локального диска, есть только один корневой каталог. А сторонние диски можно примонтировать в любом месте корнегового  и его дочерних каталогов.

```sh

```

список всех файловых систем, 
  * название без цифр - это диск
  * с цифрами - файловая система (локальный диск одного физического диска)
```sh
cat /proc/partotions | grep -v loop
```

Пример монтировани/размонтирования файловой системы sdb4
```sh
sudo /dev/sdb4 /mnt/hdd/games
sudo /mnt/hdd/games
```

Просмотр UUID файловой системы
```sh
sudo blkid
```

Автоматическое монтирование диска и монтирование ОЗУ
Получаем UUID командой blkid, открываем файл на редактирование
```sh
# открываем файл
vim /etc/fstab

# дописываем внизу
# UUID    путь монтирования     файловая система     defaults     0     0

# Монтирование устройст
sudo mount - a

# монтируем устройство в ОЗУ, пишем это в файле fstab.
# tmpfs - монтирование в озу, что монтируем, размер и mode-полные права всем пользователям
tmpfs /tmp tmpfs defaults,size=2G,mode=1777   0   0
```

## Описание каталогов корня
* / - корневой каталог
  * /boot - загрузочный каталог, содержит образы системы, а так же все, что относится к мультизагрузке
  * /sbin - бинарные файлы, который используются на ранних этапах загрузки только пользователем root
  * /lib - содержит файлы системынх библиотек, которые используются исполняемыми файлами в каталогах BIN  и SBIN
  * /etc - конфиги программ
  * /usr - пользовательское ПО, тут есть папки BIN для пользователей, SBIN для ROOT, а так же папки для библиотек. Так же есть LOCAL - в котором тоже есть BIN, SBIN и LIB, эта папка предназначена для программ созданных пользователем
  * /dev - каталог устройств
  * /proc - каталог системных файлов, они отражают состояние различных параметров системы и процессов. 
  * /sys - хранит информацию о системене от ядра
  * /opt - для установки ПО от сторонних производителей
  * /home - домашние каталоги всех пользователей
  * /root - домашний каталог пользователя ROOT
  * /run - каталог текущей информации о запущенных программах. 
  * /var - каталог служебных и временных файлов системы. Логи, базы и тд.
  * /mnt - каталог для монтирования дисков
  * /media - каталог для автоматического монтирования подключенных устройств средствами программного обеспечения UDM




