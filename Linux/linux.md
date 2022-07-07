# Содержание

[Программы](#Программы)  
* [Portainer](#Portainer)  
* [MySql](#MySql)  
* [Mosquitto](#Mosquitto)  
* [iotlink](#)  
* [](#)  
* [](#)  
* [](#)  






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
