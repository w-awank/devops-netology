# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
   > Ответ: изучил информацию на wiki по sparse файлам.

1. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
   > Ответ: фалы-хардлинки не могут иметь разные права доступа и владельца, т.к. жестко наследуют права и владельца исходного файла, на который они указывают.
   > Если поменять права или владельца у исходного файла или у его хардлинков, то права изменятся сразу на всех файлах: и на исходном, и на всех его хардлинках одинаково.

1. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
   > Ответ: Из-за ограничений Касперского не использую Vagrant. Поэтому сам руками добавил 2 диска по 2.5GB в ВМ Ubuntu на VirtualBox.
   > ```bash
   > $ lsblk | egrep 'NAME|sd[bc]'
   > NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   > sdb      8:16   0  2,5G  0 disk
   > sdc      8:32   0  2,5G  0 disk
   > ```

1. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
   > Ответ:
   > ```bash
   > $ sudo fdisk /dev/sdb
   > Команда (m для справки): g
   > Создана новая метка диска GPT (GUID: 58579B8A-356F-0F4A-B7D0-F83506BEF756).
   > Команда (m для справки): n
   > Номер раздела (1-128, по умолчанию 1): 1
   > Первый сектор (2048-5242846, по умолчанию 2048): 2048
   > Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242846, по умолчанию 5242846): +2G
   > 
   > Создан новый раздел 1 с типом 'Linux filesystem' и размером 2 GiB.
   > Команда (m для справки): n
   > Номер раздела (2-128, по умолчанию 2): 2
   > Первый сектор (4196352-5242846, по умолчанию 4196352):
   > Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242846, по умолчанию 5242846):
   > 
   > Создан новый раздел 2 с типом 'Linux filesystem' и размером 511 MiB.
   > Команда (m для справки): p
   > Диск /dev/sdb: 2,51 GiB, 2684354560 байт, 5242880 секторов
   > Disk model: VBOX HARDDISK
   > Единицы: секторов по 1 * 512 = 512 байт
   > Размер сектора (логический/физический): 512 байт / 512 байт
   > Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
   > Тип метки диска: gpt
   > Идентификатор диска: 58579B8A-356F-0F4A-B7D0-F83506BEF756
   > 
   > Устр-во     начало   Конец Секторы Размер Тип
   > /dev/sdb1     2048 4196351 4194304     2G Файловая система Linux
   > /dev/sdb2  4196352 5242846 1046495   511M Файловая система Linux
   > Команда (m для справки): w
   > Таблица разделов была изменена.
   > Вызывается ioctl() для перечитывания таблицы разделов.
   > Синхронизируются диски.
   > 
   > $ lsblk | egrep 'NAME|sdb'
   > NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   > sdb      8:16   0  2,5G  0 disk
   > ├─sdb1   8:17   0    2G  0 part
   > └─sdb2   8:18   0  511M  0 part
   > ```

1. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
   > Ответ:
   > ```bash
   > # sfdisk -d /dev/sdb | sfdisk /dev/sdc
   > Проверяется, чтобы сейчас никто не использовал этот диск... ОК
   > 
   > Диск /dev/sdc: 2,51 GiB, 2684354560 байт, 5242880 секторов
   > Disk model: VBOX HARDDISK
   > Единицы: секторов по 1 * 512 = 512 байт
   > Размер сектора (логический/физический): 512 байт / 512 байт
   > Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
   > 
   > >>> Заголовок скрипта принят.
   > >>> Заголовок скрипта принят.
   > >>> Заголовок скрипта принят.
   > >>> Заголовок скрипта принят.
   > >>> Заголовок скрипта принят.
   > >>> Заголовок скрипта принят.
   > >>> Создана новая метка диска GPT (GUID: 58579B8A-356F-0F4A-B7D0-F83506BEF756).
   > /dev/sdc1: Создан новый раздел 1 с типом 'Linux filesystem' и размером 2 GiB.
   > /dev/sdc2: Создан новый раздел 2 с типом 'Linux filesystem' и размером 511 MiB.
   > /dev/sdc3: Готово.
   > 
   > Новая ситуация:
   > Тип метки диска: gpt
   > Идентификатор диска: 58579B8A-356F-0F4A-B7D0-F83506BEF756
   > 
   > Устр-во     начало   Конец Секторы Размер Тип
   > /dev/sdc1     2048 4196351 4194304     2G Файловая система Linux
   > /dev/sdc2  4196352 5242846 1046495   511M Файловая система Linux
   > 
   > Таблица разделов была изменена
   > Вызывается ioctl() для перечитывания таблицы разделов.
   > Синхронизируются диски.
   > ```

1. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
   > Ответ:
   > ```bash
   > # mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
   > mdadm: Note: this array has metadata at the start and
   >     may not be suitable as a boot device.  If you plan to
   >     store '/boot' on this device please ensure that
   >     your boot-loader understands md/v1.x metadata, or use
   >     --metadata=0.90
   > mdadm: size set to 2094080K
   > Continue creating array? y
   > mdadm: Defaulting to version 1.2 metadata
   > mdadm: array /dev/md0 started.
   > # cat /proc/mdstat
   > Personalities : [raid1]
   > md0 : active raid1 sdc1[1] sdb1[0]
   >       2094080 blocks super 1.2 [2/2] [UU]
   > 
   > unused devices: <none>
   > ```

1. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
   > Ответ:
   > ```bash
   > # mdadm --create --verbose /dev/md1 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
   > mdadm: chunk size defaults to 512K
   > mdadm: Defaulting to version 1.2 metadata
   > mdadm: array /dev/md1 started.
   > # cat /proc/mdstat
   > Personalities : [raid1] [raid0]
   > md1 : active raid0 sdc2[1] sdb2[0]
   >       1041408 blocks super 1.2 512k chunks
   > 
   > md0 : active raid1 sdc1[1] sdb1[0]
   >       2094080 blocks super 1.2 [2/2] [UU]
   > 
   > unused devices: <none>
   > ```

1. Создайте 2 независимых PV на получившихся md-устройствах.
   > Ответ:
   > ```bash
   > # pvcreate /dev/md0
   >   Physical volume "/dev/md0" successfully created.
   > # pvcreate /dev/md1
   >   Physical volume "/dev/md1" successfully created.
   > # pvdisplay
   > "/dev/md0" is a new physical volume of "<2,00 GiB"
   > --- NEW Physical volume ---
   > PV Name               /dev/md0
   > VG Name
   > PV Size               <2,00 GiB
   > Allocatable           NO
   > PE Size               0
   > Total PE              0
   > Free PE               0
   > Allocated PE          0
   > PV UUID               LB5WkC-lh7t-dJAr-c04y-JyoG-jve0-Ac01t6
   > 
   > "/dev/md1" is a new physical volume of "1017,00 MiB"
   > --- NEW Physical volume ---
   > PV Name               /dev/md1
   > VG Name
   > PV Size               1017,00 MiB
   > Allocatable           NO
   > PE Size               0
   > Total PE              0
   > Free PE               0
   > Allocated PE          0
   > PV UUID               HZVV9x-a3ma-v9ED-YXGa-4b8c-ydER-A54r1c
   > ```

1. Создайте общую volume-group на этих двух PV.
   > Ответ:
   > ```bash
   > # vgcreate VG1 /dev/md0 /dev/md1
   >   Volume group "VG1" successfully created
   > # vgdisplay
   > --- Volume group ---
   > VG Name               VG1
   > System ID
   > Format                lvm2
   > Metadata Areas        2
   > Metadata Sequence No  1
   > VG Access             read/write
   > VG Status             resizable
   > MAX LV                0
   > Cur LV                0
   > Open LV               0
   > Max PV                0
   > Cur PV                2
   > Act PV                2
   > VG Size               <2,99 GiB
   > PE Size               4,00 MiB
   > Total PE              765
   > Alloc PE / Size       0 / 0
   > Free  PE / Size       765 / <2,99 GiB
   > VG UUID               CdyozD-6OMf-QCe7-MwYd-YMDO-k2eu-SQ53lV
   > ```

1. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
   > Ответ:
   > ```bash
   > # lvcreate -L 100M -n LV1 VG1 /dev/md1
   > Logical volume "LV1" created.
   > # lvdisplay
   >   --- Logical volume ---
   > LV Path                /dev/VG1/LV1
   > LV Name                LV1
   > VG Name                VG1
   > LV UUID                53e7RD-wZSO-AbEw-0MUa-Paps-zVM7-KepXTy
   > LV Write Access        read/write
   > LV Creation host, time wawank-VirtualBox, 2021-06-09 22:05:17 +0400
   > LV Status              available
   > # open                 0
   > LV Size                100,00 MiB
   > Current LE             25
   > Segments               1
   > Allocation             inherit
   > Read ahead sectors     auto
   > - currently set to     4096
   > Block device           253:0
   > ```

1. Создайте `mkfs.ext4` ФС на получившемся LV.
   > Ответ:
   > ```bash
   > # mkfs.ext4 /dev/VG1/LV1
   > mke2fs 1.45.5 (07-Jan-2020)
   > Creating filesystem with 25600 4k blocks and 25600 inodes
   > 
   > Allocating group tables: done
   > Сохранение таблицы inod'ов: done
   > Создание журнала (1024 блоков): готово
   > Writing superblocks and filesystem accounting information: готово
   > ```

1. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
   > Ответ:
   > ```bash
   > # mkdir /tmp/new
   > # mount /dev/VG1/LV1 /tmp/new
   > ```

1. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
   > Ответ:
   > ```bash
   > # wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
   > --2021-06-09 22:13:38--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
   > Распознаётся mirror.yandex.ru (mirror.yandex.ru)… 213.180.204.183, 2a02:6b8::183
   > Подключение к mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... соединение установлено.
   > HTTP-запрос отправлен. Ожидание ответа… 200 OK
   > Длина: 20855572 (20M) [application/octet-stream]
   > Сохранение в: «/tmp/new/test.gz»
   > 
   > /tmp/new/test.gz                        100%[============================================================================>]  19,89M  4,92MB/s    за 4,0s
   > 
   > 2021-06-09 22:13:42 (5,02 MB/s) - «/tmp/new/test.gz» сохранён [20855572/20855572]
   > ```

1. Прикрепите вывод `lsblk`.
   > Ответ:
   > ```bash
   > # lsblk
   > NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   > loop0           7:0    0 55,4M  1 loop  /snap/core18/2066
   > loop1           7:1    0 64,8M  1 loop  /snap/gtk-common-themes/1514
   > loop2           7:2    0  219M  1 loop  /snap/gnome-3-34-1804/66
   > loop3           7:3    0 55,5M  1 loop  /snap/core18/1988
   > loop4           7:4    0 65,1M  1 loop  /snap/gtk-common-themes/1515
   > loop5           7:5    0 32,1M  1 loop  /snap/snapd/11841
   > loop6           7:6    0 32,1M  1 loop  /snap/snapd/12057
   > loop7           7:7    0   51M  1 loop  /snap/snap-store/518
   > loop8           7:8    0  219M  1 loop  /snap/gnome-3-34-1804/72
   > sda             8:0    0   10G  0 disk
   > ├─sda1          8:1    0  512M  0 part  /boot/efi
   > ├─sda2          8:2    0    1K  0 part
   > └─sda5          8:5    0  9,5G  0 part  /
   > sdb             8:16   0  2,5G  0 disk
   > ├─sdb1          8:17   0    2G  0 part
   > │ └─md0         9:0    0    2G  0 raid1
   > └─sdb2          8:18   0  511M  0 part
   >   └─md1         9:1    0 1017M  0 raid0
   >     └─VG1-LV1 253:0    0  100M  0 lvm   /tmp/new
   > sdc             8:32   0  2,5G  0 disk
   > ├─sdc1          8:33   0    2G  0 part
   > │ └─md0         9:0    0    2G  0 raid1
   > └─sdc2          8:34   0  511M  0 part
   >   └─md1         9:1    0 1017M  0 raid0
   >     └─VG1-LV1 253:0    0  100M  0 lvm   /tmp/new
   > sr0            11:0    1 1024M  0 rom
   > ```

1. Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
   > Ответ:
   > ```bash
   > # gzip -t /tmp/new/test.gz
   > # echo $?
   > 0
   > ```

1. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
   > Ответ:
   > ```bash
   > # pvmove /dev/md1 /dev/md0
   >   /dev/md1: Moved: 24,00%
   >   /dev/md1: Moved: 100,00%
   > # lsblk
   > NAME          MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   > loop0           7:0    0 55,4M  1 loop  /snap/core18/2066
   > loop1           7:1    0 64,8M  1 loop  /snap/gtk-common-themes/1514
   > loop2           7:2    0  219M  1 loop  /snap/gnome-3-34-1804/66
   > loop3           7:3    0 55,5M  1 loop  /snap/core18/1988
   > loop4           7:4    0 65,1M  1 loop  /snap/gtk-common-themes/1515
   > loop5           7:5    0 32,1M  1 loop  /snap/snapd/11841
   > loop6           7:6    0 32,1M  1 loop  /snap/snapd/12057
   > loop7           7:7    0   51M  1 loop  /snap/snap-store/518
   > loop8           7:8    0  219M  1 loop  /snap/gnome-3-34-1804/72
   > sda             8:0    0   10G  0 disk
   > ├─sda1          8:1    0  512M  0 part  /boot/efi
   > ├─sda2          8:2    0    1K  0 part
   > └─sda5          8:5    0  9,5G  0 part  /
   > sdb             8:16   0  2,5G  0 disk
   > ├─sdb1          8:17   0    2G  0 part
   > │ └─md0         9:0    0    2G  0 raid1
   > │   └─VG1-LV1 253:0    0  100M  0 lvm   /tmp/new
   > └─sdb2          8:18   0  511M  0 part
   >   └─md1         9:1    0 1017M  0 raid0
   > sdc             8:32   0  2,5G  0 disk
   > ├─sdc1          8:33   0    2G  0 part
   > │ └─md0         9:0    0    2G  0 raid1
   > │   └─VG1-LV1 253:0    0  100M  0 lvm   /tmp/new
   > └─sdc2          8:34   0  511M  0 part
   >   └─md1         9:1    0 1017M  0 raid0
   > sr0            11:0    1 1024M  0 rom
   > ```

1. Сделайте `--fail` на устройство в вашем RAID1 md.
   > Ответ:
   > ```bash
   > # mdadm --fail /dev/md0 /dev/sdb1
   > mdadm: set /dev/sdb1 faulty in /dev/md0
   > ```

1. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
   > Ответ:
   > ```bash
   > # dmesg -T
   > ...
   > [Ср июн  9 22:32:35 2021] md/raid1:md0: Disk failure on sdb1, disabling device.
   >                                md/raid1:md0: Operation continuing on 1 devices.
   > ```

1. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```
   > Ответ:
   > ```bash
   > # gzip -t /tmp/new/test.gz
   > # echo $?
   > 0
   > ```

1. Погасите тестовый хост, `vagrant destroy`.
   > Ответ: Я просто выключил ВМ.
