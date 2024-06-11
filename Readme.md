Vagrant-стенд c PAM
Цель домашнего задания
Научиться создавать пользователей и добавлять им ограничения
Описание домашнего задания
1. Запретить всем пользователям кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников

* дать конкретному пользователю права работать с докером и возможность перезапускать докер сервис

1) Запуск ВМ:
stas@myserv:~/LAB/38_LDAP$ vagrant up ipa.otus.lan
2) Подключаемся к нашей созданной ВМ:
stas@myserv:~/LAB/38_LDAP$ vagrant ssh ipa.otus.lan
3) Переходим в root-пользователя:
[vagrant@ipa ~]$ sudo -i
4) Создаём пользователя otusadm и otus: 
[root@ipa ~]# sudo useradd otusadm && sudo useradd otus
5) Создаём пользователям пароли: 
[root@ipa ~]# sudo passwd otus
Changing password for user otus.
New password: 
BAD PASSWORD: The password contains the user name in some form
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@ipa ~]# sudo passwd otusadmin
passwd: Unknown user name 'otusadmin'.
[root@ipa ~]# sudo passwd otusadm
Changing password for user otusadm.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
6) Создаём группу admin: 
[root@ipa ~]# sudo groupadd -f admin
7) Добавляем пользователей vagrant,root и otusadm в группу admin:
[root@ipa ~]# usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
8) Подключаемся под созданными пользователями с хостовой машины:
stas@myserv:~$ ssh otus@192.168.57.10
(otus@192.168.57.10) Password: 
Last login: Tue Jun 11 15:27:33 2024
[otus@ipa ~]$ whoami
otus
[otus@ipa ~]$ exit
logout
Connection to 192.168.57.10 closed.
stas@myserv:~$ ssh otusadm@192.168.57.10
(otusadm@192.168.57.10) Password: 
[otusadm@ipa ~]$ whoami
otusadm
9) Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
[root@ipa ~]# cat /etc/group | grep admin
printadmin:x:994:
admin:x:1003:otusadm,root,vagrant
10)Создадим файл-скрипт /usr/local/bin/login.sh:
[root@ipa ~]# nano /usr/local/bin/login.sh
11)Добавим права на исполнение файла: 
[root@ipa ~]# chmod +x /usr/local/bin/login.sh
12) Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:
[root@ipa ~]# nano /etc/pam.d/sshd
13) В скрипт был добавлен вторник. Пробуем подключиться:
stas@myserv:~$ ssh otus@192.168.57.10
(otus@192.168.57.10) Password: 
/usr/local/bin/login.sh failed: exit code 1

(otus@192.168.57.10) Password: 
(otus@192.168.57.10) Password: 
/usr/local/bin/login.sh failed: exit code 1

(otus@192.168.57.10) Password: 
/usr/local/bin/login.sh failed: exit code 1

otus@192.168.57.10: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,keyboard-interactive).
stas@myserv:~$ ssh otusadm@192.168.57.10
(otusadm@192.168.57.10) Password: 
Last login: Tue Jun 11 15:32:48 2024 from 192.168.57.1
[otusadm@ipa ~]$ 

Пользователь otus не смог подключиться.
Пользователь otusadm смог подключиться.
