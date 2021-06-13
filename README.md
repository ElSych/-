## Курсовая работа по дисциплине информационные системы аэрокосмических комплексов.  
### Григорьев Н.С. M30-312Б-18.  

# Постановка задачи:  
1. Скачать и установить веб-сервер.  
2. Настроить его на работу с localhost  
3. Реализовать форму с загрузкой файла  

# Ход выполнения работы:  
Выполнять работу будем на Cent OS версии 7.  
Предварительно создадим пользователя "nikita" и авторизуемся.  
Создадим директорию для проекта: "mkdir ~/myproject/"  

## Шаг 1. Установка необходимых компонентов.  
Установим репозиторий EPEL, содержащий дополнительные пакеты.  
![image](https://user-images.githubusercontent.com/55539754/121819239-a62b1700-cc94-11eb-9db2-3e43008d5aee.png)

Установим pip диспетчер пакетов Python, а также файлы разработки Python, необходимые для сборки uWSGI, а также установим Nginx.  

![image](https://user-images.githubusercontent.com/55539754/121819245-b4793300-cc94-11eb-8de7-2978fb15a135.png)

## Шаг 2. Создание виртуальной среды Python.  
Для того чтобы изолировать приложение настроим виртуальную среду.  
Установим пакет virtualenv:  

![image](https://user-images.githubusercontent.com/55539754/121819257-c0fd8b80-cc94-11eb-96bf-c09049a53ebe.png)
Создадим витуальную среду:  

![image](https://user-images.githubusercontent.com/55539754/121819272-d2df2e80-cc94-11eb-9256-fbe8fec764a2.png)
Активируем её и убедимся что мы начали работу в виртуальной среде. Ввод терминала выглядит следующим образом:  

![image](https://user-images.githubusercontent.com/55539754/121819284-e12d4a80-cc94-11eb-8722-da31115a518f.png)

## Шаг 3. Создание и настройка приложения Flask.  
Установим uwsgi и flask:  

![image](https://user-images.githubusercontent.com/55539754/121819304-face9200-cc94-11eb-937f-71daa803af4a.png)
Создадим приложение Flask:  

![image](https://user-images.githubusercontent.com/55539754/121819368-56008480-cc95-11eb-9ceb-3d90f835d6ef.png)
(Исходный код приложения продемонстрирован в данном репозитории)  
Сохраним и закроем файл нажав ESC, а затем нажав Ctrl+Z,Z.  
Протестируем созданное приложение:

![image](https://user-images.githubusercontent.com/55539754/121819491-0ec6c380-cc96-11eb-9a4c-479743600156.png)


## Шаг 4. Создание точки входа WSGI.  
Создадим файл wsgi.py:  
![image](https://user-images.githubusercontent.com/55539754/121819712-3e2a0000-cc97-11eb-9db7-4ae2451e08ef.png)
Внутри напишем:  
![image](https://user-images.githubusercontent.com/55539754/121819684-1d61aa80-cc97-11eb-9df2-e9b5dda7190f.png)
## Шаг 5. Настройка конфигурации uWSGI.  
Для начала протестируем что uWSGI может обслуживать наше приложение:  

![image](https://user-images.githubusercontent.com/55539754/121819580-81d03a00-cc96-11eb-94a1-6311608c8881.png)
По указанному ранее адресу, но с портом 8000 находится содержимое html страницы.  
![image](https://user-images.githubusercontent.com/55539754/121819659-fe631880-cc96-11eb-9caf-ed0f4a7e9c41.png)

На этом работа с виртуальной средой окончена. Выйдем из нее командой deactivate:  

![image](https://user-images.githubusercontent.com/55539754/121819672-1044bb80-cc97-11eb-9caf-f0f26d402958.png)
Создадим файл конфигурации uWSGI:  
![image](https://user-images.githubusercontent.com/55539754/121819741-6f0a3500-cc97-11eb-9112-358061d3f312.png)
Внутри напишем:  
![image](https://user-images.githubusercontent.com/55539754/121819728-5732b100-cc97-11eb-9ddc-440f3bc7e97f.png)
где "module = wsgi" - исполняемый модуль, созданный ранее файл "wsgi.py";  
"master = true" - означает что uWSGI будет запускаться в главном режиме;  
"processes = 3" - будет иметь 3 рабочих процесса для обслуживания запросов;  
"socket = project.sock" - сокет, который будет использовать uWSGI;  
"chmod-socket = 660" - права на процесс uWSGI;  
"vacuum = true" - сокет будет очищен по завершении работы процесса;  

## Шаг 6. Создание файла модуля systemd.  
Создадим файл службы:  

![image](https://user-images.githubusercontent.com/55539754/121819750-7e897e00-cc97-11eb-8cef-0e950058ecb0.png)
Внутри напишем:
![image](https://user-images.githubusercontent.com/55539754/121819832-f788d580-cc97-11eb-9e37-fdb19aa96402.png)
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
![image](https://user-images.githubusercontent.com/55539754/121819838-0079a700-cc98-11eb-82a6-43e6b499f3c5.png)
## Шаг 7. Настройка Nginx.  
Теперь необходимо настроить Nginx для передачи веб-запросов в  сокет с использованием uWSGI протокола.  
Создадим и заполним новый конфигурационный файл в каталоге sites-available директории nginx  
![image](https://user-images.githubusercontent.com/55539754/121819946-9d3c4480-cc98-11eb-9903-04c40da3aa61.png)
![image](https://user-images.githubusercontent.com/55539754/121819899-5bab9980-cc98-11eb-9cc1-01f049c9fb58.png)
Закроем и сохраним файл.  
Добавим nginx пользователя в свою группу пользователей с помощью следующей команды:  

![image](https://user-images.githubusercontent.com/55539754/121819968-c1982100-cc98-11eb-96e3-9f16664ce1ec.png)
Предоставим группе пользователей права на выполнение в домашнем каталоге:  
![image](https://user-images.githubusercontent.com/55539754/121819980-d1176a00-cc98-11eb-8f05-9f69865a6817.png)
Наконец, запустим Nginx:  
![image](https://user-images.githubusercontent.com/55539754/121819956-aaf1ca00-cc98-11eb-8f29-76c81ee190f7.png)
# Вывод:  
В ходе выполнения курсовой работы было создано приложение Flask в виртуальной средe, отображающее созданную веб-страницу. 
Была создана и настроена точка входа WSGI, с помощью которой любой сервер приложений, поддерживающий WSGI, мог взаимодействовать с приложением Flask. 
Была создана служба systemd для автоматического запуска uWSGI при загрузке системы. 
Также был настроен серверный блок Nginx. 

