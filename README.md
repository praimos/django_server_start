# Настройка сервера для Django проектов (Django + Nginx + gunicorn)

## Дейсвия после установки системы (Debian / Ubuntu)

Обновление списка пакетов:
```bash
$ sudo apt update
```

Получение новых версий пакетов:
```bash
$ sudo apt upgrade -y
```

Задать пароль для root пользователя, если ранее не был заданЖ
```bash
$ sudo passwd
```

Перезагружаем севрвер.


## Установка базовых утилит и библиотек

```bash
$ sudo apt install -y vim \
	nano \
	tmux \
	htop \
	git \
	curl \
	wget \
	unzip \
	zip \
	ncdu \
	gcc \
	build-essential \
	make \
	tree \
	libssl-dev \
	zlib1g-dev \
	libbz2-dev \
	libreadline-dev \
	libsqlite3-dev \
	libncurses5-dev \
	libncursesw5-dev \
	libffi-dev \
	liblzma-dev  \
	python-pil \
	libxslt-dev \
	python-libxml2 \
	python-libxslt1 \
	libffi-dev \
	libssl-dev \
	libsqlite3-dev \
	libpq-dev \
	libxml2-dev \
	libxslt1-dev \
	libjpeg-dev \
	libfreetype6-dev \
	libcurl4-openssl-dev \
	supervisor
```
## Подключение к серверу по ключу

Для подключения к серверу с помощью ключа, создайте его на ващей машине командой (unix):
```bash
$ ssh-keygen
```
Будет сформирован каталог `.ssh` с файлами `id_rsa, id_rsa.pub`, при настройках по уумолчанию.
Для возможности подключения по ключу необходимо скопировать данные файла 
`id_rsa.pub` на удаленный серввер для вашего пользователя в файл `~/.ssh/authorized_keys`:

* Простой способ
```bash
$ ssh-copy-id username@remote_host
```
* Копироание через SSH
```bash
$ cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```
* Ручной сопособ
```bash
$ cat ~/.ssh/id_rsa.pub
# примерный вывод ключа:
# ssh-rsa AAAAB3NzaC1yc2E...9gI0x8GvaQ== demo@test

# далее на удаленном сервере
$ mkdir -p ~/.ssh
$ echo строка_публичного_ключа >> ~/.ssh/authorized_keys
$ chmod -R go= ~/.ssh
```

Отключить аутентификацию по паролю на сервере можно в файле `/etc/ssh/sshd_config`
```
...
PasswordAuthentication no
...
```

После чего необходимо перезагрузить сервер
```bash
$ sudo systemctl restart ssh
```

## Настройка SHELL на zsh (по желанию)

```bash
$ sudo apt install zsh
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

В файле `~/.zshrc` в строке `ZSH_THEME="steeef"` можно указать любюя тему из списка 
на странице https://github.com/ohmyzsh/ohmyzsh/wiki/Themes

Для активации настроек файла `~/.zshrc` выполняем команду:
```bash
$ source ~/.zshrc

or

$ . ~/.zshrc
```

## Установка нужной версии Python из исходников (на примере python3.8.2)

Я решил, что версия python из исходников будет располагаться в одной папке, 
которая будет размещяться в каталооге `/opt`

Для начала перейдем в каталог:
```bash
$ cd /opt
```

В данном каталоге все действия небходимо выполнять от пользователя `root` либо с `sudo`

Скачивваем python в текущий каталог:

```bash
$ wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tgz
```

Распаковывем архив и создаем каталог для устанавлемоего python:
```bash
$ tar xvf Python-3.8.* 
$ mkdir python3.8 
```

Конфигурируем и собираем python:
```bash
$ cd Python-3.8.2
$ ./configure --enable-optimizations --prefix=/opt/python3.8  
$ make -j8
$ sudo make altinstall
```

Обновляем pip:
```bash
$ /opt/python3.8/bin/python3.8 -m pip install -U pip  
```

Добавялем путь до нового python 
```bash
$ nano ~/.zshrc

...
export PATH=$PATH:/opt/python3.8/bin

$ . ~/.zshrc
```

## Создаем и настраиваем окружение для Django проекта

Переходим в каталог, в котором будет размещаться ваше приложение 
(у меня это домашняя диреткория)

Создаем окружение:
```bash
$ mkdir django_project
$ cd  django_project
$ python3.8 -m venv env
```

Активируем окружение и устнавливаем необходимы пакеты:
```bash
$ . ./env/bin/activate
(env)$ pip install -U pip
(env)$ pip install django gunicorn ipython
# ipython для удобства использования manage.py shell
```

После установки пакетов, стоит сформировать их список:
```bash
(env)$ pip freeze > requirements.txt
```

Также, стоит выполнять данную команду по мере установки новых пакетов.

## Создаем базовый Django проект и проверяем его работу

Создание проекта Django
```bash
(env)$ django-admin startproject test1 
```

Содержимое текущего катвлога
```bash
(env)$ ls -l
------------
drwxrwxr-x 6 alex alex 4096 апр  4 17:12 env
-rw-rw-r-- 1 alex alex  308 апр  4 17:19 requirements.txt
drwxrwxr-x 5 alex alex 4096 апр  4 20:27 test1
```

Каталог проекта
```bash
(env)$ ls -l test1
------------------
drwxrwxr-x 4 alex alex   4096 апр  4 20:27 test1
-rw-r--r-- 1 alex alex 131072 апр  4 17:53 db.sqlite3
-rwxrwxr-x 1 alex alex    626 апр  4 20:27 manage.py

(env)$ ls -l test1/test1
------------------------
-rw-rw-r-- 1 alex alex  387 апр  4 17:11 asgi.py
-rw-rw-r-- 1 alex alex    0 апр  4 17:11 __init__.py
-rw-rw-r-- 1 alex alex 3240 апр  4 20:26 settings.py
-rw-rw-r-- 1 alex alex  747 апр  4 17:11 urls.py
-rw-rw-r-- 1 alex alex  387 апр  4 17:11 wsgi.py
```

## Настройка gunicorn для запуска проекта

Создадим каталог, в который расположим скрипт запуска сервиса
```bash
(env)$ mkdir test1
----------------
(env)$ ls -l test1

drwxrwxr-x 2 alex alex   4096 апр  4 20:16 bin
drwxrwxr-x 4 alex alex   4096 апр  4 20:27 test1
-rw-r--r-- 1 alex alex 131072 апр  4 17:53 db.sqlite3
-rwxrwxr-x 1 alex alex    626 апр  4 20:27 manage.py
---------------------------------------------------
```

Создаем файл скрипта и наполняем его
```bash
(env)$ vim test1/bin/start_gunicorn.sh
---------------------------------------
#!/bin/bash
source /home/alex/django-test/env/bin/activate
exec gunicorn  -c "/home/alex/django-test/test1/test1/gunicorn_config.py" config.wsgi

(env)$ chmod u+x test1/bin/start_gunicorn.sh # даем права на исполнение
```

Конфигурируем gunicorn
```bash
(env)$ vim test1/test1/gunicorn_config.py
---------------------------------------

command = '/home/alex/django-test/env/bin/gunicorn' # путь до запуска gunicorn
pythonpath = '/home/alex/django-test/test1/test1'   # путь до конфигов проекта
bind = '127.0.0.1:8001'								# адрес для запуска сервиса
workers = 3											# количесво воркеров, 2*кол-во_процессоров+1
user = 'alex'										# от чего имени запускать провцесс
limit_request_fields = 32000						
limit_request_field_size = 0						
raw_env = 'DJANGO_SETTINGS_MODULE=test1.settings'	# файл settings проекта 
```

## Настройка supervisor для поддержки перезапуска проекта на Django

Создадим файл конфигурации в нашем проекте и наполним его
```bash
(env)$  vim  test1/test1/supervisor.test1.conf
-----------------------------------------------

environment=PYTHONPATH=/home/alex/django-test/env/bin:/home/alex/django-test/test1
command=/home/alex/django-test/test1/bin/start_gunicorn.sh
user=alex
process_name=$(program_name)s
numprocs=1
autostart=true
autorestart=true
redirect_stderr=true
```

Создаем символьную ссылку на конфиг
```bash
$ cd /etc/supervisor/conf.d
$ sudo ln -s /home/alex/django-test/test1/config/supervisor.test1.conf . 
```

Запуска сервиса
```bash
$ sudo systemctl start supervisor.service
```

## Базовая настройка Nginx

Создаем конфиг
```bash
$ vim test1/test1/nginx.conf
----------------------------

server {
        listen 80;

        index index.html index.htm;
        root /var/www/html;
        server_name _;

        location / {
                proxy_pass http://127.0.0.1:8001;
                proxy_set_header X-Forwarded-Host $server_name;
                proxy_set_header X-Real-IP $remote_addr;
                add_header P3P 'CP="ALL DCP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
                add_header Access-Control-Allow-Origin *;
        }

        location /static/ {
                root /home/alex/django-test/test1/test1/; # путь до папки со статикой, иначе работать не будет
                expires 30d;
        }
}
```

Создаем символьную ссылку на конфиг
```bash
$ cd /etc/nginx/sites-enabled
$ sudo ln -s /home/alex/django-test/test1/test1/nginx.conf . 
```

Запуска сервиса
```bash
$ sudo systemctl start nginx.service
```
