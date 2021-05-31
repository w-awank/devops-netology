# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`.
   > Ответ: это функция `chdir()`. В коде ниже это 3-я строка вывода strace по grep /tmp (последняя строка  по порядку):
   > ```bash
   > $ strace /bin/bash -c 'cd /tmp' 2>&1 | grep /tmp
   > execve("/bin/bash", ["/bin/bash", "-c", "cd /tmp"], 0x7ffdd855b600 /* 23 vars */) = 0
   > stat("/tmp", {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
   > chdir("/tmp")                           = 0
   > ```
1. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
   > Ответ: Делая strace ниже для команды file с разными файлами и грепая вывод потока ошибок (в этом потоке идет вывод strace) по функции openat видно, что идет открытие некого общего файла-ссылки /usr/share/misc/magic.mgc, который является БД команды file /usr/lib/file/magic.mgc:
   > ```bash
   > $ strace file /dev/sda 2>&1 | grep openat
   > $ strace file /dev/tty 2>&1 | grep openat
   > ...
   > openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
   > ...
   > $ ls -l /usr/share/misc/magic.mgc
   > lrwxrwxrwx 1 root root 24 мая 27 09:47 /usr/share/misc/magic.mgc -> ../../lib/file/magic.mgc
   > $ ls -l /usr/lib/file/magic.mgc
   > -rw-r--r-- 1 root root 5811536 янв 17  2020 /usr/lib/file/magic.mgc
   > ```
1. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
   > Ответ: Зная PID этого приложения с помощью `lsof -p PID` и грепая по слову deleted можно узнать номер файлового дескриптора в столбце FD. Затем просто отправляем содержимое /dev/null на этот дескриптор удаленного файла:
   > ```bash
   > $ python3 -c "import time;f=open('./do_not_delete_me','w');time.sleep(600);" &
   > [1] 49951
   > $ echo 12345 > /proc/49951/fd/3
   > $ rm do_not_delete_me
   > $ lsof -p 49951 | grep deleted
   > python3 49951 w-awank    3w   REG    8,5        6 145645 /home/w-awank/do_not_delete_me (deleted)
   > $ cat /dev/null > /proc/49951/fd/3
   > $ lsof -p 49951 | grep deleted
   > python3 49951 w-awank    3w   REG    8,5        0 145645 /home/w-awank/do_not_delete_me (deleted)
   > ```
1. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
   > Ответ: Я думая зомби-процессы занимают только RAM, т.к. они завершили работу, но по каким-то причинам их родитель PPID не смог обработать их код возврата и система поэтому не уничтожила этот процесс. И поэтому он стал зомби. 
1. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
   > Ответ: 
   > ```bash
   > # strace opensnoop-bpfcc 2>&1 | grep openat
   > openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libutil.so.1", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libexpat.so.1", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
   > openat(AT_FDCWD, "/usr/bin/pyvenv.cfg", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
   > openat(AT_FDCWD, "/usr/pyvenv.cfg", O_RDONLY) = -1 ENOENT (Нет такого файла или каталога)
   > openat(AT_FDCWD, "/etc/localtime", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/encodings/__pycache__/__init__.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/codecs.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/encodings", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/encodings/__pycache__/aliases.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/encodings/__pycache__/utf_8.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/encodings/__pycache__/latin_1.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/io.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/abc.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/site.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/os.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/stat.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/_collections_abc.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/posixpath.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/genericpath.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/_sitebuiltins.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/local/lib/python3.8/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/__pycache__/sitecustomize.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/lib/python3.8/lib-dynload", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/local/lib/python3.8/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3/dist-packages", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
   > openat(AT_FDCWD, "/usr/lib/python3/dist-packages/__pycache__/apport_python_hook.cpython-38.pyc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY|O_CLOEXEC) = 3
   > openat(AT_FDCWD, "/usr/sbin/opensnoop-bpfcc", O_RDONLY) = 3
   > ...
   > ```
1. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
   > Ответ: утилита uname использует системный вызов одноименной функции uname(). Альтернативно из man 2 узнаем, что можно посмотреть инфу в /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname} 
   > ```bash
   > $ uname -a
   > Linux wawank-VirtualBox 5.8.0-53-generic #60~20.04.1-Ubuntu SMP Thu May 6 09:52:46 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
   > $ strace uname -a
   > ...
   > uname({sysname="Linux", nodename="wawank-VirtualBox", ...}) = 0
   > ...
   > $ man 2 uname
   > ...
   > Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version, domainname}.
   > ...
   > $ cat /proc/sys/kernel/{ostype,hostname,osrelease,version,domainname}
   > Linux
   > wawank-VirtualBox
   > 5.8.0-53-generic
   > #60~20.04.1-Ubuntu SMP Thu May 6 09:52:46 UTC 2021
   > (none)
   > ```
1. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
   > Ответ: в первом случае обе команды выполнятся: вторая команда после `;` никак не зависит от результата выполнения первой команды до `;`. Во втором случае команда после `&&` выполнится только если первая команда до `&&` выполнится успешно с возвратом кода 0. Т.е. если директория /tmp/some_dir существует, то первая команда вернет код 0 и вторая команда выполнится (мы увидим `Hi`). Если же директория /tmp/some_dir не существует, то `test -d /tmp/some_dir` вернет не 0 (1) и вторая команда `echo Hi` не выполнится.
   > 
   > `set -e` останавливает выполнение если есть ошибка в команде/скрипте. Ошибка - это код возврата не 0 от выполненной команды. Есть смысл использовать set -e в скрипте только если есть всего один операнд `&&`, и в этом случае при ошибке (не 0 код выхода) в левой части от `&&` правая часть не выполнится. А вот если в скрипте с опцией set -e уже несколько строк с командами, разделенными `&&`, то это будет бессмысленно, т.к. уже при первой ошибке в команде до `&&` скрипт глобально завершится и не дойдет до проверки других сочетаний команд с `&&`.     
1. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
   > Ответ: 
   > * Указав `set -e`, скрипт немедленно завершит работу, если любая команда выйдет с ошибкой. По-умолчанию же, игнорируются любые неудачи и сценарий продолжит выполняться.
   > * Указав `set -u`, оболочка проверяет инициализацию использующихся переменных в скрипте. Если определения переменной не будет, скрипт немедленно завершится.
   > * Указав `set -x`, bash печатает в стандартный вывод все команды перед их исполнением.
   > * Указав `set -o pipefail`, bash будет проверять успешность выполнения всех команд в конвейере |, а не только самой последней как происходит по умолчанию. При ошибке в любом месте конвейера pipe эта ошибка конкретной команды будет выведена на экран, и скрипт остановит свое выполнение по строкам.
   > * Собственно поэтому параметры `set -euxo pipefail` полезно использовать в скриптах.
1. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
   > Ответ: 
   > ```bash
   > $ man ps
   > ...
   > For BSD formats and when the stat keyword is used, additional characters may be displayed:
   >   <    high-priority (not nice to other users)
   >   N    low-priority (nice to other users)
   >   L    has pages locked into memory (for real-time and custom IO)
   >   s    is a session leader
   >   l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)
   >   +    is in the foreground process group
   > ...
   > Наиболее часто встречается статус S (спящий, ожидает действия)
   > ```

 
 ---

### Как оформить ДЗ?

Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.

Также вы можете выполнить задание в [Google Docs](https://docs.google.com/document/u/0/?tgif=d) и отправить в личном кабинете на проверку ссылку на ваш документ.
Название файла Google Docs должно содержать номер лекции и фамилию студента. Пример названия: "1.1. Введение в DevOps — Сусанна Алиева"
Перед тем как выслать ссылку, убедитесь, что ее содержимое не является приватным (открыто на комментирование всем, у кого есть ссылка). 
Если необходимо прикрепить дополнительные ссылки, просто добавьте их в свой Google Docs.

Любые вопросы по решению задач задавайте в чате Slack.

---