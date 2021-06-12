## Курсовая работа по дисциплине информационные системы аэрокосмических комплексов.  
### Григорьев Н.С. M30-312Б-18.  

# Постановка задачи:  
1. Скачать и установить веб-сервер.  
2. Настроить его на работу с localhost  
3. Реализовать форму с загрузкой файла  
3.1 Захостить python приложение из предыдущего семестра, при загрузке снимка рисовать в веб карту NDVI.  

# Ход выполнения работы:  
Выполнять работу будем на Cent OS версии 7.  
Предварительно создадим пользователя "nikita" и авторизуемся.  
Создадим директорию для проекта: "mkdir ~/myproject/"  

## Шаг 1. Установка необходимых компонентов.  
Установим репозиторий EPEL, содержащий дополнительные пакеты.  

    [nikita@localhost myproject]$ sudo yum install epel-release

Установим pip диспетчер пакетов Python, а также файлы разработки Python, необходимые для сборки uWSGI, а также установим Nginx.  

    [nikita@localhost myproject]$ sudo yum install python-pip python-devel gcc nginx  

## Шаг 2. Создание виртуальной среды Python.  
Для того чтобы изолировать приложение настроим виртуальную среду.  
Установим пакет virtualenv:  

    [nikita@localhost myproject]$ sudo pip install virtualenv  
Создадим витуальную среду:  

    [nikita@localhost myproject]$ virtualenv projectenv  
Активируем её:  

    [nikita@localhost myproject]$ source projectenv/bin/activate  
Убедимся что мы начали работу в виртуальной среде. Ввод терминала выглядит следующим образом:  

    (projectenv) [nikita@localhost myproject]$  

## Шаг 3. Создание и настройка приложения Flask.  
Установим uwsgi и flask:  

    (projectenv) [nikita@localhost myproject]$ pip install uwsgi flask  
Создадим приложение Flask:  

    (projectenv) [nikita@localhost myproject]$ vi ~/myproject/app.py  
(Исходный код приложения продемонстрирован в данном репозитории)  
Сохраним и закроем файл нажав ESC, а затем нажав Ctrl+Z,Z.  
Протестируем созданное приложение. Для этого запустим его в фоновом режиме:  

    (projectenv) [nikita@localhost myproject]$ python ~/myproject/app.py &  
> *Serving Flask app 'app' (lazy loading)  
> *Environment: prodction  
> *Debug mode: on  
> *Running on all addresses.  
> *Running on http://10.0.15:5000 (Press CTRL+C) to quit)  

А затем обратимся к содержимому с помощью curl. Выведем первые 8 строчек html страницы: 

    (projectenv) [nikita@localhost project]$ curl -L http://10.0.2.15:5000 | head -n 8  

> \<!DOCTYPE html>  
>  \<html lang="ru">  
>  \<head>  
>   \<meta charset="UTF-8">  
>   \<title>Image processing</title>  
>  \</head>  
> \<body>  
>   \<p class="fadein1">Загрузите изображения для красного и инфракрасного спектров в формате tif:</p>  

После этого остановим Flask приложение с помощью fg:  

    (projectenv) [nikita@localhost myproject]$ fg  
    python app.py  
    ^C  

## Шаг 3. Создание точки входа WSGI.  
Создадим файл wsgi.py:  

    (projectenv) [nikita@localhost myproject]$ vi ~/myproject/wsgi.py  
Внутри напишем:  

    from myproject import app  
    if __name__ == "__main__":  
      app.run()  

## Шаг 4. Настройка конфигурации uWSGI.  
Для начала протестируем что uWSGI может обслуживать наше приложение:  

    (projectenv) [nikita@localhost myproject]$ uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi &  
Убедимся что по указанному ранее адресу, но с портом 8000 находится содержимое html страницы.  

    (projectenv) [nikita@localhost myproject]$ curl -L http://10.0.15:8000 | head -n 5  

> \<!DOCTYPE html>  
> \<html lang="ru">  
> \<head>  
> \<meta charset="UTF-8">  
> \<title>Image processing</title>  

После этого приостановим uwsgi:  

    (projectenv) [nikita@localhost myproject]$ fg  
    uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi
    ^C  
На этом работа с виртуальной средой окончена. Выйдем из нее командой deactivate:  

    (projectenv) [nikita@localhost myproject]$ deactivate  
Создадим файл конфигурации uWSGI:  

    [nikita@localhost myproject]$ vi ~/myproject/project.ini  
Внутри напишем:  

    [uwsgi]  
    module = wsgi  
    master = true  
    processes = 3  
    socket = project.sock  
    chmod-socket = 660  
    vacuum = true  
где "module = wsgi" - исполняемый модуль, созданный ранее файл "wsgi.py";  
"master = true" - означает что uWSGI будет запускаться в главном режиме;  
"processes = 3" - будет иметь 3 рабочих процесса для обслуживания запросов;  
"socket = project.sock" - сокет, который будет использовать uWSGI;  
"chmod-socket = 660" - права на процесс uWSGI;  
"vacuum = true" - сокет будет очищен по завершении работы процесса;  

## Шаг 5. Создание файла модуля systemd.  
Создадим файл службы:  

    [nikita@localhost myproject]$ sudo vi /etc/systemd/system/myproject.service  
Внутри напишем:

    [Unit]  
    Description=uWSGI for myproject  
    After=network.target  
    [Service]  
    User=nikita  
    Group=nginx  
    WorkingDirectory=/home/nikita/myproject  
    Environment="PATH=/home/nikita/myproject/projectenv/bin"  
    ExecStart=/home/nikita/myproject/projectenv/bin/uwsgi --ini project.ini  
    [Install]  
    WantedBy=multi-user.target  
где "[Unit]", "[Service]", "[Install]" - заголовки разделов;  
"Description" - описание службы;  
"After" - цель, по достижении которой будет производиться запуск;  
"User" - пользователь;  
"Group" - группа;  
"WorkingDirectory" - рабочая директория в которой хранятся исполняемые файлы;  
"Environment" - директория виртуальной среды;  
"ExecStart" - команда для запуска процесса;  
"WantedBy" - когда запускаться службе.  
Запустим созданную службу:  

    [nikita@localhost myproject]$ sudo systemctl start project  
    [nikita@localhost myproject]$ sudo systemctl enable project  

## Шаг 6. Настройка Nginx.  
Теперь необходимо настроить Nginx для передачи веб-запросов в  сокет с использованием uWSGI протокола.  
Откроем файл конфигурации Nginx:  

    [nikita@localhost myproject]$ sudo vi /etc/nginx/nginx.conf  
Найдем блок server {} в теле http: 

    ...  
      http {  
      ...  
        include /etc/nginx/conf.d/*.conf;  
        *
        server {  
            listen 80 default_server;  
            }  
      ...  
      }  
    ...  

Выше него (*) создадим свой:  

    server {  
      listen 80;  
      server_name 10.0.2.15;  
      
      location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/nikita/myproject/myproject.sock;
    }
Закроем и сохраним файл.  
Добавим nginx пользователя в свою группу пользователей с помощью следующей команды:  

    [nikita@localhost myproject]$ sudo usermod -a -G nikita nginx  
Предоставим группе пользователей права на выполнение в домашнем каталоге:  

    [nikita@localhost myproject]$ chmod 710 /home/nikita  
Наконец, запустим Nginx:  

    [nikita@localhost myproject]$ sudo systemctl start nginx  
    [nikita@localhost myproject]$ sudo systemctl enable nginx  

# Вывод:  
В ходе выполнения курсовой работы было создано приложение Flask в виртуальной средe, позволяющее загружать файлы формата ".tif" и получать цветное изображение NDVI. 
Была создана и настроена точка входа WSGI, с помощью которой любой сервер приложений, поддерживающий WSGI, мог взаимодействовать с приложением Flask. 
Была создана служба systemd для автоматического запуска uWSGI при загрузке системы. 
Также был настроен серверный блок Nginx. 

