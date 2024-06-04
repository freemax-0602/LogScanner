**Домашнее задание №10**

**Тема** ***"Работа с загрузчиком"***

**Задача**
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig или в /etc/default).
2. Установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).
3. Дополнить unit-файл httpd (он же apache2) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.

**Результат работы**

***Задание 1***
1. Поднят Vagrantfile для проведения экспериментов
2. Создан bash-скрипт `watchlog.sh`, который ищет данные по ключевоу слову в log-файле
3. Создан конфиг `watchlog` для сервиса
4. Создан тестовый log-файл
5. Создан unit для сервиса `watchlog.service`
6. Создан unit для таймера `watchlog.timer`
7. Запущен сервис и проверен анализ лога:
```
[root@demon ~]# systemctl start watchlog.service
[root@demon ~]# systemctl status watchlog.service
● watchlog.service - My watchlog service
   Loaded: loaded (/etc/systemd/system/watchlog.service; static; vendor preset: disabled)
   Active: inactive (dead) since Tue 2024-06-04 11:16:47 UTC; 5s ago
  Process: 2958 ExecStart=/opt/watchlog.sh $WORD $LOG (code=exited, status=0/SUCCESS)
 Main PID: 2958 (code=exited, status=0/SUCCESS)

Jun 04 11:16:47 demon systemd[1]: Starting My watchlog service...
Jun 04 11:16:47 demon systemd[1]: Started My watchlog service.
[root@demon ~]# tail -f /var/log/messages 
Jun  4 11:04:59 demon systemd-logind: Removed session 2.
Jun  4 11:05:01 demon systemd-logind: Removed session 1.
Jun  4 11:05:01 demon systemd: Removed slice User Slice of vagrant.
Jun  4 11:05:17 demon systemd: Created slice User Slice of vagrant.
Jun  4 11:05:17 demon systemd: Started Session 3 of user vagrant.
Jun  4 11:05:17 demon systemd-logind: New session 3 of user vagrant.
Jun  4 11:05:40 demon systemd: Started Run watchlog script every 30 second.
Jun  4 11:16:47 demon systemd: Starting My watchlog service...
Jun  4 11:16:47 demon root: Tue Jun  4 11:16:47 UTC 2024: I found word, Master!
Jun  4 11:16:47 demon systemd: Started My watchlog service.

```

***Задание 2***
1. Установлены необходимые пакеты 
`yum install epel-release -y && yum install spawn-fcgi php php-cli
mod_fcgid httpd -y
`
2. Раскоментированы строки с переменными в конфиге сервиса spawn `/etc/sysconfig/spawn-fcgi`
```
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"
```

3. Добавлен unit для spawn - `spawn-fcgi.service` :
```
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target

```

4. Проверена работа сервиса:
```
[root@demon ~]# systemctl start spawn-fcgi           
[root@demon ~]# systemctl status spawn-fcgi
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2024-06-04 12:34:02 UTC; 4s ago
 Main PID: 3802 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─3802 /usr/bin/php-cgi
           ├─3803 /usr/bin/php-cgi
           ├─3804 /usr/bin/php-cgi
           ├─3805 /usr/bin/php-cgi
           ├─3806 /usr/bin/php-cgi
           ├─3807 /usr/bin/php-cgi
           ├─3808 /usr/bin/php-cgi
           ├─3809 /usr/bin/php-cgi
           ├─3810 /usr/bin/php-cgi
           ├─3811 /usr/bin/php-cgi
           ├─3812 /usr/bin/php-cgi
           ├─3813 /usr/bin/php-cgi
           ├─3814 /usr/bin/php-cgi
           ├─3815 /usr/bin/php-cgi
           ├─3816 /usr/bin/php-cgi
           ├─3817 /usr/bin/php-cgi
           ├─3818 /usr/bin/php-cgi
           ├─3819 /usr/bin/php-cgi
           ├─3820 /usr/bin/php-cgi
           ├─3821 /usr/bin/php-cgi
           ├─3822 /usr/bin/php-cgi
           ├─3823 /usr/bin/php-cgi
           ├─3824 /usr/bin/php-cgi
           ├─3825 /usr/bin/php-cgi
           ├─3826 /usr/bin/php-cgi
           ├─3827 /usr/bin/php-cgi
           ├─3828 /usr/bin/php-cgi
           ├─3829 /usr/bin/php-cgi
           ├─3830 /usr/bin/php-cgi
           ├─3831 /usr/bin/php-cgi
           ├─3832 /usr/bin/php-cgi
           ├─3833 /usr/bin/php-cgi
           └─3834 /usr/bin/php-cgi

Jun 04 12:34:02 demon systemd[1]: Started Spawn-fcgi startup service by Otus.
```