## Краткая информация по настройке vsftpd с виртуальными аккаунтами
## На протяжении довольно долгого времени подобная конфигурация использовалась мною в различных проектах, поэтому заслужила быть зафиксированной здесь.

#### Установка основных компонент на хосте:
```
yum -y install vsftpd
yum -y install ftp
yum -y install compat-db

mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
vi /etc/vsftpd/vsftpd.conf

anonymous_enable=NO
local_enable=YES
chroot_local_user=YES
sudo service vsftpd restart
chkconfig vsftpd on
```

или
```
systemctl start vsftpd
systemctl enable vsftpd
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload

setsebool -P ftpd_full_access 1
```
##### Управление виртуальными пользователями:
##### Создадим в системе пользователя и группу virtual:

```
groupadd virtual
useradd virtual -g virtual
cat /etc/passwd | grep virtual:

virtual:x:1001:1001:,,,:/home/ftp/internal:/bin/false
cat /etc/group | grep virtual

virtual:x:1001:oracle,virtual
```

##### Корневая папка для всех вирт. пользователей: 
```
mkdir /home/ftp/internal
chown virtual:virtual -R /home/ftp 
chmod -w /home/ftp/internal
mkdir /home/ftp/internal/master
```
##### Изменим файл /etc/pam.d/vsftpd
```
cat vsftpd

auth     required  /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
account  required  /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
session    required     pam_loginuid.so
```
##### Создаем папку с настройками для каждого виртуального пользователя:
```
mkdir /etc/vsftpd/vsftpd_user_conf
```
##### Создаем в этой папке профиль для пользователя:
```
cat /etc/vsftpd/vsftpd_user_conf/master

 anon_world_readable_only=NO
 anon_umask=022
 write_enable=YES
 anon_upload_enable=YES
 anon_other_write_enable=YES
 anon_mkdir_write_enable=YES
 anon_max_rate=0
 local_root=/home/ftp/internal/
```
##### Скрипт для создания базы логин-пароль
```
cat makedb.sh

#!/bin/bash
db_load -T -t hash -f users_db vsftpd_login.db
```
##### Файл с именами и паролями пользователей
```
cat users_db

master
Margo
user
pass4user
```
#### Настройки в /etc/vsftpf/vsftpd.conf
```
# =========================================================================
# VSFTPD CONFIGURATION FILE (by TIMIJAN)
# =========================================================================
listen=YES
listen_port=21
anonymous_enable=NO
local_enable=YES
write_enable=NO

anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
anon_world_readable_only=NO
chroot_local_user=YES
use_localtime=YES

#ENABLE VIRTUAL USERS
guest_enable=YES
guest_username=virtual

#PAM
pam_service_name=vsftpd

dirmessage_enable=YES
xferlog_enable=YES
xferlog_std_format=NO
#log_ftp_protocol=YES
xferlog_file=/var/log/vsftpd.log

#connect_from_port_20=YES
pasv_min_port=50000
pasv_max_port=65535
user_config_dir=/etc/vsftpd/vsftpd_user_conf
async_abor_enable=YES

#SPEED AND TIME SETTINGS FOR SESSION
anon_max_rate=512000
idle_session_timeout=600
data_connection_timeout=320
ftpd_banner=_devhost_ftp_server_
```
