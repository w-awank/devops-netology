# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки"

## Обязательные задания

1. Есть скрипт:
	```bash
	a=1
	b=2
	c=a+b
	d=$a+$b
	e=$(($a+$b))
	```
	* Какие значения переменным c,d,e будут присвоены?
	* Почему?
	> Решение.
    >```bash
    > $ echo $c
    > a+b # Переменная $c объявлена неявно и по умолчанию это строка. Поэтому ее значение есть строка из 3-х символов. Символы a и b написаны без $. Поэтому при формировании переменной $c эти символы и знак + остались как есть строкой в выражении присваивания. 
    > $ echo $d
    > 1+2 # То же самое, только в выражение присваивания вместо символов a и b подставили их строковые значения через знак $ (1 и 2 соответственно ). Поэтому получилась строка 1+2.
    > $ echo $e
    > 3 # Наконец-то получили сумму чисел из-за применения конструкции $((...)) арифметической операции сложения значений переменных a и b.
    >```
   
1. На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным. В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
	```bash
	while ((1==1)
	do
	curl https://localhost:4757
	if (($? != 0))
	then
	date >> curl.log
	fi
	done
	```
    > Решение. В данном скрипте 3 ошибки: забыли 2-ую закрывающуюся круглую скобку в строке while, использовали дополнение лога >>, а не перезапись >, а также не написали break для выхода из бесконечного цикла при успешном поднятии сервиса. Вот код правильного скрипта:
    > ```bash
    > #!/usr/bin/env bash
    > while ((1==1))
    > do
    > curl https://localhost:4757 1>/dev/null 2>&1
    > if (($? != 0))
    > then
    > echo Service is DOWN
    > date > curl.log
    > else
    > echo Service is OK 
    > break
    > fi
    > sleep 5
    > done
    > ``` 
1. Необходимо написать скрипт, который проверяет доступность трёх IP: 192.168.0.1, 173.194.222.113, 87.250.250.242 по 80 порту и записывает результат в файл log. Проверять доступность необходимо пять раз для каждого узла.
   > Решение. Выбрал другие доступные по TCP 80 IP.
   > ```bash
   > #!/usr/bin/env bash
   > array_int=(0 1 2 3 4)
   > array_hosts=(192.168.88.11 95.173.128.90 194.105.131.10)
   > 
   > echo Checking HTTP availability of three servers: ${array_hosts[0]}, ${array_hosts[1]}, ${array_hosts[2]}
   > for i in ${array_int[@]}
   > do
   >     for j in ${!array_hosts[@]}
   >     do
   >         curl -m 10 http://${array_hosts[$j]} 1>/dev/null 2>&1
   >         if (($? == 0))
   >         then
   >             date >> 3ipstatus.log
   >             echo ${array_hosts[$j]} is OK by TCP 80 >> 3ipstatus.log
   >         else
   >             date >> 3ipstatus.log
   >             echo ${array_hosts[$j]} is DOWN by TCP 80 >> 3ipstatus.log
   >         fi
   >     done
   >     sleep 5
   > done
   > cat 3ipstatus.log
   > echo See log in file 3ipstatus.log
   > ```

1. Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается
   > Решение.
   > ```bash
   > #!/usr/bin/env bash
   > array_hosts=(192.168.88.11 95.173.128.90 194.105.131.10)
   > 
   > echo Checking HTTP availability of three servers: ${array_hosts[0]}, ${array_hosts[1]}, ${array_hosts[2]}
   > while ((1==1))
   > do
   >     for j in ${!array_hosts[@]}
   >     do
   >         curl -m 10 http://${array_hosts[$j]} 1>/dev/null 2>&1
   >         if (($? == 0))
   >         then
   >             date >> 3ipstatus.log
   >             echo ${array_hosts[$j]} is OK by TCP 80 >> 3ipstatus.log
   >         else
   >             date >> 3iperror.log
   >             echo ${array_hosts[$j]} is DOWN by TCP 80 >> 3iperror.log
   >             cat 3iperror.log
   >             echo See log in file 3ipstatus.log
   >             echo See error in file 3iperror.log
   >             exit 1
   >         fi
   >     done
   >     sleep 5
   > 
   > done
   > cat 3ipstatus.log
   > echo See log in file 3ipstatus.log
   > echo See error in file 3iperror.log
   > ```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Мы хотим, чтобы у нас были красивые сообщения для коммитов в репозиторий. Для этого нужно написать локальный хук для git, который будет проверять, что сообщение в коммите содержит код текущего задания в квадратных скобках и количество символов в сообщении не превышает 30. Пример сообщения: \[04-script-01-bash\] сломал хук.
> Решение. Написал bash хук commit-msg и положил в .git/hooks/, предварительно дав права на выполнение `chmod 755 .git/hooks/commit-msg`. Проверял работу - все ок. Коммиты с сообщением, не соответствующем шаблону `[<цифра><цифра>-<тект>-<цифра><цифра>-<текст>]*` и/или более 30 символов НЕ принимаются гитом.
> ```bash
> $ cat .git/hooks/commit-msg
> #!/usr/bin/env bash
> commitregexp="^\[[0-9]{2}-\w+-[0-9]{2}-\w+\].*"
> 
> egrep -q $commitregexp .git/COMMIT_EDITMSG
> if (($? == 0))
> then
>     commitmsg=`cat .git/COMMIT_EDITMSG`
>     lengthcommitmsg=${#commitmsg}
>     if (($lengthcommitmsg <= 30))
>     then
>         exit 0
>     else
>         echo "Your commit message is more than 30 chars! Please change commit message."
>         exit 1
>     fi
> else
>     echo "Your commit message is not according company policy! Please change commit message."
>     exit 1
> fi
> ```