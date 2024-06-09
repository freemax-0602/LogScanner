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

***Задание 3***
1. Отредактирован шаблон для httpd
```
EnvironmentFile=/etc/sysconfig/httpd-%I
```
2. Созданы файлы окружения:
```
[root@demon ~]# cat /etc/sysconfig/httpd-first 
#
# This file can be used to set additional environment variables for
# the httpd process, or pass additional options to the httpd
# executable.
#
# Note: With previous versions of httpd, the MPM could be changed by
# editing an "HTTPD" variable here.  With the current version, that
# variable is now ignored.  The MPM is a loadable module, and the
# choice of MPM can be changed by editing the configuration file
# /etc/httpd/conf.modules.d/00-mpm.conf.
# 

#
# To pass additional options (for instance, -D definitions) to the
# httpd binary at startup, set OPTIONS here.
#
OPTIONS=-f conf/first.conf

#
# This setting ensures the httpd process is started in the "C" locale
# by default.  (Some modules will not behave correctly if
# case-sensitive string comparisons are performed in a different
# locale.)
#
LANG=C
[root@demon ~]# cat /etc/sysconfig/httpd-second 
#
# This file can be used to set additional environment variables for
# the httpd process, or pass additional options to the httpd
# executable.
#
# Note: With previous versions of httpd, the MPM could be changed by
# editing an "HTTPD" variable here.  With the current version, that
# variable is now ignored.  The MPM is a loadable module, and the
# choice of MPM can be changed by editing the configuration file
# /etc/httpd/conf.modules.d/00-mpm.conf.
# 

#
# To pass additional options (for instance, -D definitions) to the
# httpd binary at startup, set OPTIONS here.
#
OPTIONS=-f conf/second.conf

#
# This setting ensures the httpd process is started in the "C" locale
# by default.  (Some modules will not behave correctly if
# case-sensitive string comparisons are performed in a different
# locale.)
#
LANG=C
[root@demon ~]# 
```

4. Созданы файлы конфигов для окружений:
```
[root@demon ~]# cat /etc/httpd/conf/first.conf | grep Listen
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on specific IP addresses as shown below to 
#Listen 12.34.56.78:80
Listen 80


[root@demon ~]# cat /etc/httpd/conf/second.conf | grep Listen
# Listen: Allows you to bind Apache to specific IP addresses and/or
# Change this to Listen on specific IP addresses as shown below to 
#Listen 12.34.56.78:80
Listen 8080
```

5. Для автоматической конфигурации хост-машины, создан playbook `Configure_Systemd`.
   В  Vagrantfile добавлся опция provisioning:
   ```
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "Configure_Systemd.yml"
    end
   ```
6. После команды `vagrant up` автоматически запустится плейбук, и установит наши сервисы:
  ```
  [root@demon ~]#
[root@demon ~]# systemctl status watchlog.timer
● watchlog.timer - Run watchlog script every 30 second
   Loaded: loaded (/etc/systemd/system/watchlog.timer; enabled; vendor preset: disabled)
   Active: active (waiting) since Sun 2024-06-09 20:51:20 UTC; 14min ago

Jun 09 20:51:20 demon systemd[1]: Started Run watchlog script every 30 second.
[root@demon ~]# tail -f /var/log/messages
Jun  9 21:01:01 localhost systemd: Started Session 5 of user root.
Jun  9 21:05:02 localhost systemd: Created slice User Slice of vagrant.
Jun  9 21:05:02 localhost systemd: Started Session 6 of user vagrant.
Jun  9 21:05:02 localhost systemd-logind: New session 6 of user vagrant.
Jun  9 21:05:21 localhost systemd: Starting My watchlog service...
Jun  9 21:05:21 localhost root: Sun Jun  9 21:05:21 UTC 2024: I found word, Master!
Jun  9 21:05:21 localhost systemd: Started My watchlog service.
Jun  9 21:05:35 localhost systemd: Starting My watchlog service...
Jun  9 21:05:35 localhost root: Sun Jun  9 21:05:35 UTC 2024: I found word, Master!
Jun  9 21:05:35 localhost systemd: Started My watchlog service.

[root@demon ~]# systemctl status spawn-fcgi.service
● spawn-fcgi.service - Spawn-fcgi startup service by Otus
   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2024-06-09 20:58:50 UTC; 7min ago
 Main PID: 5661 (php-cgi)
   CGroup: /system.slice/spawn-fcgi.service
           ├─5661 /usr/bin/php-cgi
           ├─5664 /usr/bin/php-cgi
           ├─5665 /usr/bin/php-cgi
           ├─5666 /usr/bin/php-cgi
           ├─5667 /usr/bin/php-cgi
           ├─5668 /usr/bin/php-cgi
           ├─5669 /usr/bin/php-cgi
           ├─5670 /usr/bin/php-cgi
           ├─5671 /usr/bin/php-cgi
           ├─5672 /usr/bin/php-cgi
           ├─5673 /usr/bin/php-cgi
           ├─5674 /usr/bin/php-cgi
           ├─5675 /usr/bin/php-cgi
           ├─5676 /usr/bin/php-cgi
           ├─5677 /usr/bin/php-cgi
           ├─5678 /usr/bin/php-cgi
           ├─5679 /usr/bin/php-cgi
           ├─5680 /usr/bin/php-cgi
           ├─5681 /usr/bin/php-cgi
           ├─5682 /usr/bin/php-cgi
           ├─5683 /usr/bin/php-cgi
           ├─5684 /usr/bin/php-cgi
           ├─5685 /usr/bin/php-cgi
           ├─5686 /usr/bin/php-cgi
           ├─5687 /usr/bin/php-cgi
           ├─5688 /usr/bin/php-cgi
           ├─5689 /usr/bin/php-cgi
           ├─5690 /usr/bin/php-cgi
           ├─5691 /usr/bin/php-cgi
           ├─5692 /usr/bin/php-cgi
           ├─5693 /usr/bin/php-cgi
           ├─5694 /usr/bin/php-cgi
           └─5695 /usr/bin/php-cgi

Jun 09 20:58:50 demon systemd[1]: Started Spawn-fcgi startup service by Otus.


[root@demon ~]# systemctl status httpd@first.service
● httpd@first.service - The Apache HTTP Server
   Loaded: loaded (/etc/systemd/system/httpd@.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2024-06-09 20:58:54 UTC; 8min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 6438 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/system-httpd.slice/httpd@first.service
           ├─6438 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─6439 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─6440 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─6441 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─6442 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           ├─6443 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
           └─6444 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

Jun 09 20:58:54 demon systemd[1]: Starting The Apache HTTP Server...
Jun 09 20:58:54 demon httpd[6438]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'Ser...is message
Jun 09 20:58:54 demon systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
  ```
