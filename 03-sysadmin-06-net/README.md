# Домашнее задание к занятию "3.6. Компьютерные сети, лекция 1"

1. Необязательное задание:
можно посмотреть целый фильм в консоли `telnet towel.blinkenlights.nl` :)
   > Решение. Посмотрел бы, но сервер похоже канул в небытие :)

1. Узнайте о том, сколько действительно независимых (не пересекающихся) каналов есть в разделяемой среде WiFi при работе на 2.4 ГГц. Стандарты с полосой 5 ГГц более актуальны, но регламенты на 5 ГГц существенно различаются в разных странах, а так же не раз обновлялись. В качестве дополнительного вопроса вне зачета, попробуйте найти актуальный ответ и на этот вопрос.
   > Решение. Действительно непересекающимися в 2.4ГГц являются всего 3 канала – это каналы 1, 6 и 11 (для ширины канала 20 МГц), с шириной канала 40 МГц – это каналы 3 и 11. В частотном диапазоне 5 ГГц доступно 23 неперекрывающихся канала по 20 МГц. Здесь уже можно использовать не только ширину 20/40 МГц, но и широкий канал в 80 МГц (основной + вспомогательный).
   
1. Адрес канального уровня – MAC адрес – это 6 байт, первые 3 из которых называются OUI – Organizationally Unique Identifier или уникальный идентификатор организации. Какому производителю принадлежит MAC `38:f9:d3:55:55:79`?
   > Решение. Это Apple Inc. (38:f9:d3).

1. Каким будет payload TCP сегмента, если Ethernet MTU задан в 9001 байт, размер заголовков IPv4 – 20 байт, а TCP – 32 байта?
   > Решение. Payload TCP = TCP MSS = 9001-20-32=8949 байт. 

1. Может ли во флагах TCP одновременно быть установлены флаги SYN и FIN при штатном режиме работы сети? Почему да или нет?
   > Решение. Не могут в штатном режиме в tcp сегменте одновременно быть установлены флаги SYN и FIN, т.к. это взаимоисключающие состояния соединения tcp (запрос на установление соединения и запрос на закрытие соединения). 

1. `ss -ula sport = :53` на хосте имеет следующий вывод:

   ```bash
   State           Recv-Q          Send-Q                   Local Address:Port                     Peer Address:Port          Process
   UNCONN          0               0                        127.0.0.53%lo:domain                        0.0.0.0:*
   ```

   Почему в `State` присутствует только `UNCONN`, и может ли там присутствовать, например, `TIME-WAIT`?
   > Решение. Этой командой мы отобразили в системе все прослушиваемые UDP сокеты, отфильтровав по source port udp 53 (Это DNS). TIME-WAIT в State у UDP не может присутствовать, т.к. для UDP нет процедуры завершения соединения. В данный момент состояние UNCONN говорит, что нет активной передачи данных по этому source port (peer address отсутствует). 

7. Обладая знаниями о том, как штатным образом завершается соединение (FIN от инициатора, FIN-ACK от ответчика, ACK от инициатора), опишите в каких состояниях будет находиться TCP соединение в каждый момент времени на клиенте и на сервере при завершении. Схема переходов состояния соединения вам в этом поможет.
   > Решение. Инициатор, отправив FIN, переходит из состояния ESTABLISHED в FIN-WAIT-1. Ответчик, получив от инициатора FIN, отправляет в ответ инициатору ACK, переходя сам из состояния ESTABLISHED в CLOSE-WAIT. Инициатор, получив ACK от ответчика, переходит из состояния FIN-WAIT-1 в FIN-WAIT-2. Ответчик теперь со своей стороны отправляет FIN инициатору, переходя из состояния CLOSE-WAIT в LAST-ACK. Инициатор, получив FIN от ответчика, переходит из FIN-WAIT-2 в TIME-WAIT и отправляет последний ACK ответчику. После этого и инициатор, и ответчик переходят в состояние CLOSED.  

1. TCP порт – 16 битное число. Предположим, 2 находящихся в одной сети хоста устанавливают между собой соединения. Каким будет теоретическое максимальное число соединений, ограниченное только лишь параметрами L4, которое параллельно может установить клиент с одного IP адреса к серверу с одним IP адресом? Сколько соединений сможет обслужить сервер от одного клиента? А если клиентов больше одного?
   > Решение. Теоретическое максимальное число соединений, ограниченное только лишь параметрами L4, которое параллельно может установить клиент с одного IP адреса к серверу с одним IP адресом - 65535. Аналогично сервер сможет обслужить до 65535 соединений от одного клиента (с одного клиентского ip). Если клиентов больше одного, то сервер уже может обслуживать больше 65535 соединений, т.к. они будут инициированы с различных ip клиентов. И тут уже сервер ограничивается по соединениям только своими ресурсами CPU/RAM.  

1. Может ли сложиться ситуация, при которой большое число соединений TCP на хосте находятся в состоянии  `TIME-WAIT`? Если да, то является ли она хорошей или плохой? Подкрепите свой ответ пояснением той или иной оценки.
   > Решение. Думаю такая ситуация маловероятна, т.к. это предпоследнее состояние инициатора при завершении соединения TCP, переходящее затем в CLOSED после определенного времени без ожидания получения каких-либо других подтверждений от ответчика. Если такая ситуация могла бы сложиться, то это однозначно плохо, т.к. не освобождался бы source/destination сокет (порт) на инициаторе. И соответственно после этого инициатор смог бы установить меньшее число новых подключений TCP из-за неосвобожденных сокетов TIME-WAIT.

1. Чем особенно плоха фрагментация UDP относительно фрагментации TCP?
   > Решение. На уровне IP при сборке фрагментированных пакетов для UDP наверное может и не собраться исходный IP пакет из-за негарантированной доставки UDP... Плюс скорострельность UDP начинает страдать при его фрагментации на уровне IP. И весь смысл использования быстрого UDP теряется, т.к. нужно время на фрагментацию и сборку. Наверное лучше не использовать большие датаграммы UDP, чтобы не пришлось их фрагментировать на уровне IP.      

1. Если бы вы строили систему удаленного сбора логов, то есть систему, в которой несколько хостов отправяют на центральный узел генерируемые приложениями логи (предположим, что логи – текстовая информация), какой протокол транспортного уровня вы выбрали бы и почему? Проверьте ваше предположение самостоятельно, узнав о стандартном протоколе syslog.
   > Решение. Я бы выбрал TCP для контроля целостности и гарантированного получения текстовых логов. На удивление syslog работает на UDP/514. Скорее всего клиент и сервер syslog на прикладном уровне сами обеспечивают гарантированную доставку текстовых логов. И для syslog важна скорость доставки и небольшие размеры UDP, которые везде пролезут.

1. Сколько портов TCP находится в состоянии прослушивания на вашей виртуальной машине с Ubuntu, и каким процессам они принадлежат?
   > Решение.
   > ```bash
   > $ ss -tle
   > State          Recv-Q         Send-Q                 Local Address:Port                     Peer Address:Port         Process
   > LISTEN         0              4096                   127.0.0.53%lo:domain                        0.0.0.0:*             uid:101 ino:20430 sk:280 <->
   > LISTEN         0              128                          0.0.0.0:ssh                           0.0.0.0:*             ino:25762 sk:281 <->
   > LISTEN         0              5                          127.0.0.1:ipp                           0.0.0.0:*             ino:24043 sk:282 <->
   > LISTEN         0              4096                               *:9100                                *:*             ino:24346 sk:283 v6only:0 <->
   > LISTEN         0              128                             [::]:ssh                              [::]:*             ino:25773 sk:284 v6only:1 <->
   > LISTEN         0              5                              [::1]:ipp                              [::]:*             ino:24042 sk:285 v6only:1 <->
   > $ sudo lsof -ni :domain
   > COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   > systemd-r 508 systemd-resolve   12u  IPv4  20429      0t0  UDP 127.0.0.53:domain
   > systemd-r 508 systemd-resolve   13u  IPv4  20430      0t0  TCP 127.0.0.53:domain (LISTEN)
   > $ sudo lsof -ni :ssh
   > COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   > sshd     688    root    3u  IPv4  25762      0t0  TCP *:ssh (LISTEN)
   > sshd     688    root    4u  IPv6  25773      0t0  TCP *:ssh (LISTEN)
   > $ sudo lsof -ni :ipp
   > COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   > cupsd     637 root    6u  IPv6  24042      0t0  TCP [::1]:ipp (LISTEN)
   > cupsd     637 root    7u  IPv4  24043      0t0  TCP 127.0.0.1:ipp (LISTEN)
   > cups-brow 652 root    7u  IPv4  25419      0t0  UDP *:631
   > $ sudo lsof -ni :9100
   > COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   > node_expo 569 root    3u  IPv6  24346      0t0  TCP *:9100 (LISTEN)
   > ```

1. Какой ключ нужно добавить в `tcpdump`, чтобы он начал выводить не только заголовки, но и содержимое фреймов в текстовом виде? А в текстовом и шестнадцатиричном?
   > Решение. Ключ `-A` для вывода содержимого фреймов в текстовом виде. Ключ `-X` или `-XX` для вывода содержимого фреймов в текстовом и HEX виде.

1. Попробуйте собрать дамп трафика с помощью `tcpdump` на основном интерфейсе вашей виртуальной машины и посмотреть его через tshark или Wireshark (можно ограничить число пакетов `-c 100`). Встретились ли вам какие-то установленные флаги Internet Protocol (не флаги TCP, а флаги IP)? Узнайте, какие флаги бывают. Как на самом деле называется стандарт Ethernet, фреймы которого попали в ваш дамп? Можно ли где-то в дампе увидеть OUI?
   > Решение. Встретились флаги IPv4 0x40 Don't fragment, запрещающие разбивать IP пакет на фрагменты при отправке на роутер. Всего есть 3 бита флагов IPv4. Нулевой бит зарезервирован и должен быть всегда равен нулю. Если значение первого бита ноль, то допускается фрагментация пакетов, если единица (бит DF или Do not Fragment), то устройства компьютерной сети не будут выполнять фрагментацию. Второй бит служит для того, чтобы конечные узлы понимали, где начинается последовательность фрагментированных пакетов, а где она заканчивается, если значение этого бита равно единице (MF More Fragments), то узел понимает, что этот пакет не последний и нужно ждать еще пакеты, чтобы собрать изначально разделенный пакет. Стандарт Ethernet называется IEEE 802.3, обозначающий проводной Ethernet. В дампе практически везде MAC адреса отправителя и получателя идут с расшифрованной информацией производителя OUI.     
   > ```bash
   > # tcpdump -i enp0s3 -c 100 -w /root/my_pcap.pcap
   > ```
