**Домашнее задание №10**

**Тема** ***"Работа с загрузчиком"***

**Задача**
1. Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig или в /etc/default).
2. Установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).
3. Дополнить unit-файл httpd (он же apache2) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.

**Результат работы**
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