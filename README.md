# OTUS ДЗ №23.  Сбор и анализ логов. #
-----------------------------------------------------------------------
## Домашнее задание ##

1. В вагранте поднимаем 2 машины web и log
2. на web поднимаем nginx
3. на log настраиваем центральный лог сервер на любой системе на выбор (journald, rsyslog, elk)
4. настраиваем аудит, следящий за изменением конфигов нжинкса
5. Все критичные логи с web должны собираться и локально и удаленно.
6. Все логи с nginx должны уходить на удаленный сервер (локально только критичные).
7. Логи аудита должны также уходить на удаленную систему.

Задачи со звёдочкой(если я правильно понял задание):
1. развернуть еще машину elk, таким образом настроить 2 центральных лог системы elk и какую либо еще;
2. в elk должны уходить только логи нжинкса, во вторую систему все остальное.

-----------------------------------------------------------------------
## Результат ##

1. Поднимаем виртуальные машины командой ```vagrant up```

2. Заходим на машину web и выполняем следующие команды:
```
vagrant ssh web
sudo -i
logger -p local7.crit "Test critical log"
curl localhost
curl localhost/qqqq
```
3. В логе видим запись:
```
[root@web ~]# tail /var/log/boot.log 
Nov 13 09:01:18 web vagrant: Test critical log
```

4. Для проверки работы аудита изменений, вносим изменение в файл /etc/nginx/nginx.conf

5. Подключаемся к машине log:
```
vagrant ssh log
sudo -i
```
6. Проверяем наличие записи в логе
```
[root@log ~]# tail /var/log/rsyslog/web/vagrant.log 
Nov 13 09:01:18 web vagrant: Test critical log
```
7. Проверяем работу аудита изменений:
```
[root@log ~]# ausearch -k nginx_watch
----
time->Sun Nov 13 08:57:13 2022
node=web type=CONFIG_CHANGE msg=audit(1668329833.260:1066): auid=4294967295 ses=4294967295 subj=system_u:system_r:unconfined_service_t:s0 op=add_rule key="nginx_watch" list=4 res=1
----
time->Sun Nov 13 09:02:00 2022
node=web type=PROCTITLE msg=audit(1668330120.042:1111): proctitle=7669002F6574632F6E67696E782F6E67696E782E636F6E66
node=web type=PATH msg=audit(1668330120.042:1111): item=1 name="/etc/nginx/.nginx.conf.swp" inode=11595 dev=08:01 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 objtype=CREATE cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PATH msg=audit(1668330120.042:1111): item=0 name="/etc/nginx/" inode=85 dev=08:01 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=CWD msg=audit(1668330120.042:1111):  cwd="/root"
node=web type=SYSCALL msg=audit(1668330120.042:1111): arch=c000003e syscall=2 success=yes exit=4 a0=7a2d90 a1=c2 a2=180 a3=69676e2f6374652f items=2 ppid=3676 pid=3701 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vi" exe="/usr/bin/vi" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_watch"
----
time->Sun Nov 13 09:02:00 2022
node=web type=PROCTITLE msg=audit(1668330120.042:1112): proctitle=7669002F6574632F6E67696E782F6E67696E782E636F6E66
node=web type=PATH msg=audit(1668330120.042:1112): item=1 name="/etc/nginx/.nginx.conf.swx" inode=11596 dev=08:01 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 objtype=CREATE cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PATH msg=audit(1668330120.042:1112): item=0 name="/etc/nginx/" inode=85 dev=08:01 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=CWD msg=audit(1668330120.042:1112):  cwd="/root"
node=web type=SYSCALL msg=audit(1668330120.042:1112): arch=c000003e syscall=2 success=yes exit=5 a0=7a2d60 a1=c2 a2=180 a3=69676e2f6374652f items=2 ppid=3676 pid=3701 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vi" exe="/usr/bin/vi" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_watch"
----
time->Sun Nov 13 09:02:00 2022
node=web type=PROCTITLE msg=audit(1668330120.042:1113): proctitle=7669002F6574632F6E67696E782F6E67696E782E636F6E66
node=web type=PATH msg=audit(1668330120.042:1113): item=1 name="/etc/nginx/.nginx.conf.swx" inode=11596 dev=08:01 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 objtype=DELETE cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PATH msg=audit(1668330120.042:1113): item=0 name="/etc/nginx/" inode=85 dev=08:01 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=CWD msg=audit(1668330120.042:1113):  cwd="/root"
node=web type=SYSCALL msg=audit(1668330120.042:1113): arch=c000003e syscall=87 success=yes exit=0 a0=7a2d60 a1=7ffc5a296890 a2=7ffc5a296890 a3=7ffc5a296220 items=2 ppid=3676 pid=3701 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vi" exe="/usr/bin/vi" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_watch"
----
time->Sun Nov 13 09:02:00 2022
node=web type=PROCTITLE msg=audit(1668330120.042:1114): proctitle=7669002F6574632F6E67696E782F6E67696E782E636F6E66
node=web type=PATH msg=audit(1668330120.042:1114): item=1 name="/etc/nginx/.nginx.conf.swp" inode=11595 dev=08:01 mode=0100600 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:httpd_config_t:s0 objtype=DELETE cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=PATH msg=audit(1668330120.042:1114): item=0 name="/etc/nginx/" inode=85 dev=08:01 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:httpd_config_t:s0 objtype=PARENT cap_fp=0000000000000000 cap_fi=0000000000000000 cap_fe=0 cap_fver=0
node=web type=CWD msg=audit(1668330120.042:1114):  cwd="/root"
node=web type=SYSCALL msg=audit(1668330120.042:1114): arch=c000003e syscall=87 success=yes exit=0 a0=7a2d90 a1=7ffc5a296890 a2=7ffc5a296890 a3=7ffc5a296220 items=2 ppid=3676 pid=3701 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="vi" exe="/usr/bin/vi" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_watch"
```
