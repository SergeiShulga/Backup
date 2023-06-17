### 10.4 «Резервное копирование»- Сергей Шульга

#### Задание 1
В чём разница между:

* полным резервным копированием,
* дифференциальным резервным копированием,
* инкрементным резервным копированием.
Приведите ответ в свободной форме.

#### Полное резервное копирование (Full Backup)
Создается полная копия набора данных. С точки зрения скорости восстановления и уровня надежности считается наиболее выигрышным вариантом бэкапа. Однако у полного резервного копирования есть и минусы — full backup требует много времени для создания копии и создает существенную нагрузку на сеть.

#### Плюсы:
* высокая скорость восстановления данных;
* высокий уровень надежности — все данные в одном бэкапе;
* простота управления.
#### Минусы:
* необходимо много места для хранения копий;
* высокая нагрузка на сеть;
* требует много времени для создания бэкапа.

#### Дифференциальное резервное копирование (Differential Backup)
Подразумевает создание резервной копии, которая включает только те данные, которые были изменены с момента создания предыдущего полного бэкапа.

#### Плюсы:
* более высокая скорость создания копий по сравнению с full backup;
* по скорости восстановления быстрее инкрементного, но медленнее полного бэкапа;
* по надежности выигрывает у инкрементного бэкапа.
#### Минусы:
* каждый новый дифференциальный бэкап требует создается дольше и требует больше места, чем предыдущий.

#### Инкрементальное резервное копирование (Incremental Backup)
При использовании этого метода резервная копия будет содержать только изменения, сделанные с момента последнего резервного копирования.

#### Плюсы:
* высокая скорость создания копий;
* копии занимают мало места;
* бэкапы занимают мало места;
* не создает высокой нагрузки на сеть.
Минусы:
* трудоемкость восстановления данных;
* риск неудачного восстановления при повреждении какого-либо сегмента из цепочки бэкапа.

#### Задание 2
Установите программное обеспечении Bacula, настройте bacula-dir, bacula-sd, bacula-fd. Протестируйте работу сервисов.

Пришлите:
- конфигурационные файлы для bacula-dir, bacula-sd, bacula-fd,
- скриншот, подтверждающий успешное прохождение резервного копирования.

### bacula-sd
```
Storage {                             # definition of myself
  Name = debian-sd
  SDPort = 9103                  # Director's port
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/run/bacula"
  Plugin Directory = "/usr/lib/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = 127.0.0.1
}

Director {
  Name = debian-dir
  Password = "8z3RrpNnb6Hp2Np-sjsZBZc4o0NvDnawP"
}

Director {
  Name = debian-mon
  Password = "7SZq6_RoaK-SHiiMvC6iEQLHcwFOJ8ywL"
  Monitor = yes
}

Autochanger {
  Name = FileChgr1
  Device = FileChgr1-Dev1
  Changer Command = ""
  Changer Device = /dev/null
}

Device {
  Name = FileChgr1-Dev1
  Media Type = File1
  Archive Device = ./bacula/restore
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
  Maximum Concurrent Jobs = 5
}

Messages {
  Name = Standard
  director = debian-dir = all
}
```
### bacula-fd
```
Director {
  Name = debian-dir
  Password = "SoLEcBu7BMI7vIUpG1viKvnFbbT0gKqL-"
}

FileDaemon {                          # this is me
  Name = debian-fd
  FDport = 9102                  # where we listen for the director
  WorkingDirectory = /var/lib/bacula
  Pid Directory = /run/bacula
  Maximum Concurrent Jobs = 20
  Plugin Directory = /usr/lib/bacula
  FDAddress = 127.0.0.1
}
Messages {
  Name = Standard
  director = debian-dir = all, !skipped, !restored
}
```
![alt text](https://github.com/SergeiShulga/Backup/blob/main/img/002.png)


#### Задание 3
Установите программное обеспечении Rsync. Настройте синхронизацию на двух нодах. Протестируйте работу сервиса.
Пришлите рабочую конфигурацию сервера и клиента Rsync блоком кода в вашем md-файле.

### Клиент
```
pid file = /var/run/rsyncd.pid
log file = /var/log/rsync.log
transfer logging = true
munge symlinks = yes

[data]
path = /root/
uid = root
read only = yes
list = yes
comment = Data backup Dir ][a][a][a
auth users = backup
secrets file = /etc/rsyncd.scrt

[new]
path = /etc/
uid = root
read only = yes
list = yes
comment = System fileset
auth users = backup
secrets file = /etc/rsyncd.scrt
```
### Сервер
```
#!/bin/bash
date
syst_dir=/backup/
srv_name=debian10
srv_ip=192.168.1.92
srv_user=backup

srv_dir=data
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"

srv_dir=new
echo "Start backup ${srv_name}"
# Создаем папку для инкрементных бэкапов
mkdir -p ${syst_dir}${srv_name}/increment/
/usr/bin/rsync -avz --progress --delete --password-file=/etc/rsyncd.scrt ${srv_user}@${srv_ip}::${srv_dir} ${syst_dir}${srv_name}/current/ --backup --backup-dir=${syst_dir}${srv_name}/increment/`date +%Y-%m-%d`/
/usr/bin/find ${syst_dir}${srv_name}/increment/ -maxdepth 1 -type d -mtime +30 -exec rm -rf {} \;
date
echo "Finish backup ${srv_name}"
```
![alt text](https://github.com/SergeiShulga/Backup/blob/main/img/003.png)

Задание со звёздочкой*
Это задание дополнительное. Его можно не выполнять. На зачёт это не повлияет. Вы можете его выполнить, если хотите глубже разобраться в материале.

Задание 4*
Настройте резервное копирование двумя или более методами, используя одну из рассмотренных команд для папки /etc/default. Проверьте резервное копирование.

Пришлите рабочую конфигурацию выбранного сервиса по поставленной задаче.
