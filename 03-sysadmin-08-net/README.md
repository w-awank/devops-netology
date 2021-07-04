# Домашнее задание к занятию "3.8. Компьютерные сети, лекция 3"

1. ipvs. Если при запросе на VIP сделать подряд несколько запросов (например, `for i in {1..50}; do curl -I -s 172.28.128.200>/dev/null; done `), ответы будут получены почти мгновенно. Тем не менее, в выводе `ipvsadm -Ln` еще некоторое время будут висеть активные `InActConn`. Почему так происходит?
   > Решение. Если речь идет про режим работы DR (Direct Routing) балансировщика ipvs, который как раз мы рассматривали в лекции, то так происходит потому, что балансировщик ipvs не видит прямые ответы backend серверов клиентам, т.к. они идут напрямую клиентам мимо балансировщика. Поэтому он (ipvs) не знает - ответил backend клиенту на запрос или нет. Из-за этого TCP соединение на нем еще висит какое-то время. Эти соединения исчезают с балансировщика после определенного тайм-аута. 

1. На лекции мы познакомились отдельно с ipvs и отдельно с keepalived. Воспользовавшись этими знаниями, совместите технологии вместе (VIP должен подниматься демоном keepalived). Приложите конфигурационные файлы, которые у вас получились, и продемонстрируйте работу получившейся конструкции. Используйте для директора отдельный хост, не совмещая его с риалом! Подобная схема возможна, но выходит за рамки рассмотренного на лекции.
   > Решение. Я подготовил 5 ВМ: w-awank-cl(192.168.88.11), w-awank-lb1(192.168.88.12), w-awank-lb2(192.168.88.13), w-awank-rs1(192.168.88.14), w-awank-rs2(192.168.88.15). На балансировщики установил keepalived, а на реалы - nginx. Я выбрал режим работы DR для keepalived, чтобы избежать использования NAT и iptables на балансировщиках. На реалах в конфигурации ядра как в лекции задал параметры, чтобы ядро не обрабатывало ARP на loopback интерфейсах, а также задал VIP как алиас с маской /32 на loopback интерфейсах реалов.
   > ```bash
   > root@w-awank-lb1:/home/w-awank# cat /etc/keepalived/keepalived.conf
   > vrrp_script chk_keepalived {
   >     script "systemctl status keepalived"
   >     interval 2
   > }
   > 
   > vrrp_instance VI_1 {
   >     state MASTER
   >     interface enp0s3
   >     virtual_router_id 20
   >     priority 100
   >     advert_int 1
   >     authentication {
   >         auth_type PASS
   >         auth_pass netology_secret
   >     }
   >     virtual_ipaddress {
   >         192.168.88.20/24 dev enp0s3
   >     }
   >     track_script {
   >         chk_keepalived
   >     }
   > }
   > 
   > virtual_server 192.168.88.20 80 {
   >     protocol TCP
   >     delay_loop 10
   >     lvs_sched rr
   >     lvs_method DR
   > 
   >     real_server 192.168.88.14 80 {
   >         TCP_CHECK {
   >             connect_timeout 3
   >         }
   >     }
   > 
   >     real_server 192.168.88.15 80 {
   >         TCP_CHECK {
   >             connect_timeout 3
   >         }
   >     }
   > }
   > # root@w-awank-lb1:/home/w-awank# systemctl enable keepalived
   > # root@w-awank-lb1:/home/w-awank# systemctl start keepalived
   > ``` 
   > На 2-ом балансировщике все то же самое, только в файле keepalived.conf state BACKUP.
   > ```bash
   > # root@w-awank-lb1:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > TCP  192.168.88.20:80 rr
   >   -> 192.168.88.14:80             Route   1      0          0
   >   -> 192.168.88.15:80             Route   1      0          0
   >
   > # root@w-awank-lb2:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > TCP  192.168.88.20:80 rr
   >   -> 192.168.88.14:80             Route   1      0          0
   >   -> 192.168.88.15:80             Route   1      0          0
   > ```
   > Конфигурируем ядро на реалах для отключения обработки ARP на loopback:
   > ```bash
   > # root@w-awank-rs1:/home/w-awank# echo "net.ipv4.conf.all.arp_ignore=1" >>/etc/sysctl.d/99-sysctl.conf
   > # root@w-awank-rs1:/home/w-awank# echo "net.ipv4.conf.all.arp_announce=2" >>/etc/sysctl.d/99-sysctl.conf
   > # root@w-awank-rs1:/home/w-awank# sysctl -w net.ipv4.conf.all.arp_ignore=1
   > net.ipv4.conf.all.arp_ignore = 1
   > # root@w-awank-rs1:/home/w-awank# sysctl -w net.ipv4.conf.all.arp_announce=2
   > net.ipv4.conf.all.arp_announce = 2
   > 
   > # root@w-awank-rs2:/home/w-awank# echo "net.ipv4.conf.all.arp_ignore=1" >>/etc/sysctl.d/99-sysctl.conf
   > # root@w-awank-rs2:/home/w-awank# echo "net.ipv4.conf.all.arp_announce=2" >>/etc/sysctl.d/99-sysctl.conf
   > # root@w-awank-rs2:/home/w-awank# sysctl -w net.ipv4.conf.all.arp_ignore=1
   > net.ipv4.conf.all.arp_ignore = 1
   > # root@w-awank-rs2:/home/w-awank# sysctl -w net.ipv4.conf.all.arp_announce=2
   > net.ipv4.conf.all.arp_announce = 2
   > ```
   > Задаем VIP /32 на loopback на реалах:
   > ```bash
   > # root@w-awank-rs1:/home/w-awank# ip addr add 192.168.88.20/32 dev lo label lo:20
   > # root@w-awank-rs2:/home/w-awank# ip addr add 192.168.88.20/32 dev lo label lo:20
   > ```
   > Проверяем наш стэнд с клиента:
   > ```bash
   > w-awank@w-awank-cl:~$ for i in {1..50}; do curl -I -s http://192.168.88.20; done
   > ```
   > Получив 50 ответов в консоль клиента, проверяем балансировку. Балансировка отлично работает с активным балансером на lb1:
   > ```bash
   > # root@w-awank-lb1:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > TCP  192.168.88.20:80 rr
   >   -> 192.168.88.14:80             Route   1      0          25
   >   -> 192.168.88.15:80             Route   1      0          25
   > 
   > # root@w-awank-lb2:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > TCP  192.168.88.20:80 rr
   >   -> 192.168.88.14:80             Route   1      0          0
   >   -> 192.168.88.15:80             Route   1      0          0
   > 
   > # root@w-awank-rs1:/home/w-awank# wc -l /var/log/nginx/access.log
   > 25 /var/log/nginx/access.log
   > # root@w-awank-rs2:/home/w-awank# wc -l /var/log/nginx/access.log
   > 25 /var/log/nginx/access.log
   > ```
   > Проверим работу резервного lb2 в активном режиме, остановив демон keepalived на lb1. Тоже все отлично:
   > ```bash
   > # root@w-awank-lb1:/home/w-awank# systemctl stop keepalived
   > # root@w-awank-lb1:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > 
   > w-awank@w-awank-cl:~$ for i in {1..50}; do curl -I -s http://192.168.88.20; done
   > 
   > # root@w-awank-lb2:/home/w-awank# ipvsadm -ln
   > IP Virtual Server version 1.2.1 (size=4096)
   > Prot LocalAddress:Port Scheduler Flags
   >   -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
   > TCP  192.168.88.20:80 rr
   >   -> 192.168.88.14:80             Route   1      0          25
   >   -> 192.168.88.15:80             Route   1      0          25
   > 
   > # root@w-awank-rs1:/home/w-awank# wc -l /var/log/nginx/access.log
   > 50 /var/log/nginx/access.log
   > # root@w-awank-rs2:/home/w-awank# wc -l /var/log/nginx/access.log
   > 50 /var/log/nginx/access.log
    > ```

1. В лекции мы использовали только 1 VIP адрес для балансировки. У такого подхода несколько отрицательных моментов, один из которых – невозможность активного использования нескольких хостов (1 адрес может только переехать с master на standby). Подумайте, сколько адресов оптимально использовать, если мы хотим без какой-либо деградации выдерживать потерю 1 из 3 хостов при входящем трафике 1.5 Гбит/с и физических линках хостов в 1 Гбит/с? Предполагается, что мы хотим задействовать 3 балансировщика в активном режиме (то есть не 2 адреса на 3 хоста, один из которых в обычное время простаивает).
   > Решение. С учетом вопроса и ответов https://netology.ru/profile/program/devsys-9/lessons/78183/lesson_items/377058 думаю что 4-х VIP будет достаточно. На самом деле не очень понятно как правильно ответить на этот вопрос. Еще хотелось бы увидеть реальную конфигурацию файлов keepalived.conf в случае 3-х активных балансировщиков... 
