Настраиваем центральный сервер для сбора логов journald
--------
### Разворачиваем три сервера при помощи Vagrant+Ansible:

Реализация отправки дублированных логов с systemd-journal(web01) через systemd-journal-upload.service на systemd-journal-remote(logsrv(/var/log/journal/remote/)):

```
# Пакет systemd-journal-gateway включает в себя:
# systemd-journal-gatewayd — http-демон, открывающий порт для просмотра (или скачивания) записей журнала
# systemd-journal-remote — демон, скачивающий или принимающий записи журналов на центральном сервере
# systemd-journal-upload — демон, дублирующий записи журналов на удаленный сервер
```

web01
- навешиваем роль journaldClient(Сервер)
  - systemd-journal-upload (для дублирования бинарных логов SystemD) [journal-upload.conf.d/client.conf](https://github.com/kyourselfer/OTUS_LinuxAdmin201804/blob/master/lesson21_Journald_ELK/roles/journaldClient/files/client_journald_upload.conf)
  
logsrv
 - навешиваем роль journaldSrv
   - systemd-journal-remote (пасивный режим - без опроса клиентов)
```
---
- hosts: WebServers logsrv 
  become: yes
  roles:
    - { role: CentOS_Srv }

- hosts: web01
  become: yes
  roles:
    - { role: nginx }
    - { role: journaldClient }

- hosts: logsrv
  become: yes
  roles:
    - { role: ntpd }
    - { role: journaldSrv }
```
![journald](https://github.com/kyourselfer/OTUS_LinuxAdmin201804/blob/master/lesson21_Journald_ELK/img/journald.gif)