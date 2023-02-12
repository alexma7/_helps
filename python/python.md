# Содержание <span id="contents"><span>
* [Скачивание пакетов](#download_packages)

* [Установка окружения и пакетов](#install_env_packages)
* [Установка расширений2](#install_packeges2)




## Скачивание пакетов <span id="install_packeges">[&uarr;](#download_packages)<span> 
- Перейдем в папку с пакетами
```
    cd /d D:\other\python\311
```
- Для скачивания последних версий 
```
    pip download -r .\requirements.txt -d .\ 
```
- Для установки 
```
pip install --no-index --find-links .\ -r requirements.txt
```
- Для скачивание WHL-пакетов и используем команду **pip download -r D:\python_settings\requirements.txt -d D:\python_settings\ex**. 
- Для скачивания файлов определенной верссии, надо указать еще ее, иногда качается предыдущая **pip download --only-binary=:all: --python-version=311 --abi=cp311 --platform=win_amd64 -r D:\other\python\311\requirements.txt -d D:\other\python\311**




## Установка окружения и пакетов <span id="install_env_packages">[&uarr;](#contents)<span> 
- После того, как вы впервые скачали репозиторий с фласком, необходимо в эту папку установить виртуальное окружение. Все пакеты находятся в **D:\other\python\3.10**

- откроем консоль win+r вводим cmd и жмем enter
- в консоли вводим cd /d G:\K309\python\3.11
- Переходим в папку с проектом, либо туда, где будет лежать папка с виртуальным окружением. У меня это: **cd /d C:\progs\git\test_flask**
- Создаем виртуальное окружение **py -m venv interpreter_flask**
- Далее, надо активировать виртуалку (надо делать все время перед использованием, что бы подпись interpreter_flask была слева) вводим **.\interpreter_flask\Scripts\activate.bat** Можно просто зайти в папку проекта -> scripts и перетащить файл activate.bat в консоль CMD.
- Устанавливаем пакеты через файл **requirements_flask_local** командой: **pip install -r D:\other\python\3.10\requirements_flask_local.txt --no-index --find-links D:\other\python\3.10**
- - если перейдейдем в папку с пакетами и requirements, то команда будет следующая:
    ```

    pip install -r requirements.txt --no-index --find-links .\
    ```
- Если нужно добавить какие-то пакеты, то просто дописываем их в файл requirements_flask_local вставляем в папку недостающие файлы и запускаем верхнюю команду.





