---
## Front matter
title: "Лабораторная работа № 5. Дискреционное разграничение прав в Linux. Исследование влияния дополнительных атрибутов"
subtitle: "Дисциплина: Информационная безопасность"
author: "Манаева Варвара Евгеньевна"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Техническое оснащение:

- Персональный компьютер с операционной системой Windows 10;
- Планшет для записи видеосопровождения и голосовых комментариев;
- Microsoft Teams, использующийся для записи скринкаста лабораторной работы;
- Приложение Pycharm для редактирования файлов формата *md*;
- *pandoc* для конвертации файлов отчётов и презентаций.

# Цели и задачи работы
## Цель

Изучение механизмов изменения идентификаторов, применения SetUID- и Sticky-битов. Получение практических навыков работы
в консоли с дополнительными атрибутами. Рассмотрение работы механизма смены идентификатора процессов пользователей, а также
влияние бита Sticky на запись и удаление файлов.

## Задачи

- Настроить по заданию лабораторный стенд;
- Создать программы;
- Подробно разобрать влияние Sticky-бита на удаление и запись файлов.

# Теоретическое введение

Setuid – это бит разрешения, который позволяет пользователю запускать исполняемый файл с правами владельца этого файла.
Другими словами, использование этого бита позволяет нам поднять привилегии пользователя в случае, если это необходимо.
Классический пример использования этого бита в операционной системе это команда sudo. Принцип работы Setgid очень похож 
на setuid с отличием, что файл будет запускаться пользователем от имени группы, которая владеет файлом.

Последний специальный бит разрешения – это `Sticky Bit`. В случае, если этот бит установлен для папки, то файлы в этой папке
могут быть удалены только их владельцем. Пример использования этого бита в операционной системе это системная папка `/tmp`.
Эта папка разрешена на запись любому пользователю, но удалять файлы в ней могут только пользователи, являющиеся владельцами этих файлов.

# Выполнение лабораторной работы [@lab:5]
## Создание программ

1. Войдите в систему от имени пользователя guest.

![](image/1_1.png){#fig:101 width=80%}

2. Создайте программу simpleid.c:

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
int
main ()
{
    uid_t uid = geteuid ();
    gid_t gid = getegid ();
    printf ("uid=%d, gid=%d\n", uid, gid);
    return 0;
}
```

![](image/1_1.png){#fig:101 width=80%}

3. Скомплилируйте программу и убедитесь, что файл программы создан, командой `gcc simpleid.c -o simpleid`
4. Выполните программу simpleid командой `./simpleid`
5. Выполните системную программу id `id`



При использовании команды `id`, в отличие от использования написаной нами программы, выводится 

6. Усложните программу, добавив вывод действительных идентификаторов и получившуюся программу назовите `simpleid2.c`.

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
int
main ()
{
    uid_t real_uid = getuid();
    uid_t e_uid = geteuid();
    gid_t real_gid = getgid();
    gid_t e_gid = getegid() ;
    printf ("e_uid=%d, e_gid=%d\n", e_uid, e_gid);
    printf ("real_uid=%d, real_gid=%d\n", real_uid, real_gid);
    return 0;
}
```



7. Скомпилируйте и запустите `simpleid2.c` командами:

```
gcc simpleid2.c -o simpleid2
./simpleid2
```



8. От имени суперпользователя выполните команды:

```
chown root:guest /home/guest/simpleid2
chmod u+s /home/guest/simpleid2
```



9. Используйте sudo или повысьте временно свои права с помощью su и выполните эти команды.



Команда `chown root:guest /home/guest/simpleid2` меняет владельца каталога на `root`, а группу каталога меняет на группу
`guest` (вернее, оставляет без изменений). Команда chmod `u+s /home/guest/simpleid2` даёт права на выполнение от имени 
суперпользователя.

10. Выполните проверку правильности установки новых атрибутов и смены владельца файла simpleid2 командой `ls -l simpleid2`



11. Запустите simpleid2 и id 

```
id
./simpleid2
```


Команда `id`, в отличие от написаной программы, выводит 

12. Проделайте тоже самое относительно SetGID-бита.

```
chown root:root /home/guest/simpleid2
chmod g+s /home/guest/simpleid2
id
./simpleid2
```



13. Создайте программу `readfile.c`

```c
#include <fcntl.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
int
main (int argc, char* argv[])
{
    unsigned char buffer[16];
    size_t bytes_read;
    int i;
    int fd = open (argv[1], O_RDONLY);
    do
    {
        bytes_read = read (fd, buffer, sizeof (buffer));
        for (i =0; i < bytes_read; ++i) printf("%c", buffer[i]);
    }
    while (bytes_read == sizeof (buffer));
    close (fd);
    return 0;
}
```

14. Откомпилируйте её командой `gcc readfile.c -o readfile`



15. Смените владельца у файла readfile.c (или любого другого текстового файла в системе) и измените права так, чтобы 
только суперпользователь (root) мог прочитать его, a guest не мог.

```
chmod 770 /home/guest/readfile.c
chmod g+s /home/guest/readfile.c
```



16. Проверьте, что пользователь guest не может прочитать файл `readfile.c` командой `cat /home/guest/readfile.c` 



17. Смените у программы readfile владельца и установите SetU’D-бит.



18. Проверьте, может ли программа readfile прочитать файл readfile.c?
19. Проверьте, может ли программа readfile прочитать файл /etc/shadow?
Отразите полученный результат и ваши объяснения в отчёте.

## Исследование Sticky-бита

1. Выясним, установлен ли атрибут `Sticky` на директории `/tmp` командой `ls -l / | grep tmp`

![](image/2_1.png){#fig:201 width=80%}

2. От имени пользователя guest создадим файл `file01.txt` в директории `/tmp`со словом test командой `echo "test" > /tmp/file01.txt`

![](image/2_2.png){#fig:202 width=80%}

3. Просмотрим атрибуты у только что созданного файла и разрешим чтение и запись для категории пользователей «все остальные» командами:
 
```
ls -l /tmp/file01.txt
chmod o+rw /tmp/file01.txt
ls -l /tmp/file01.txt
```

![](image/2_3.png){#fig:203 width=80%}

4. От пользователя `guest2` (не являющегося владельцем) попробуем прочитать файл `/tmp/file01.txt` командой `cat /tmp/file01.txt`

![](image/2_4.png){#fig:204 width=80%}

5. От пользователя `guest2` попробуем дозаписать в файл `/tmp/file01.txt` слово test2 командой `echo "test2" > /tmp/file01.txt`

![](image/2_5.png){#fig:205 width=80%}

Операцию выполнить (НЕ) удалось.

6. Проверим содержимое файла командой `cat /tmp/file01.txt`

![](image/2_6.png){#fig:206 width=80%}

7. От пользователя guest2 попробуем записать в файл `/tmp/file01.txt` слово test3, стерев при этом всю имеющуюся в файле информацию командой
`echo "test3" > /tmp/file01.txt`

![](image/2_7.png){#fig:207 width=80%}

Операцию выполнить (НЕ) удалось.

8. Проверим содержимое файла командой `cat /tmp/file01.txt`

![](image/2_8.png){#fig:208 width=80%}

9. От пользователя guest2 попробуем удалить файл `/tmp/file01.txt` командой `rm /tmp/fileOl.txt`

![](image/2_9.png){#fig:209 width=80%}

Файл удалить (НЕ) удалось.

10. Повысим свои права до суперпользователя следующей командой `su -` и выполним после этого команду, снимающую 
атрибут `t` (Sticky-бит) с директории `/tmp`, --- `chmod -t /tmp`

![](image/2_10.png){#fig:210 width=80%}

11. Покинем режим суперпользователя командой `exit`

![](image/2_11.png){#fig:211 width=80%}

12. От пользователя `guest2` проверим, что атрибута `t` у директории `/tmp` нет командой `ls -l / | grep tmp`

![](image/2_12.png){#fig:212 width=80%}

13. Повторим шаги 4-9.

![](image/2_13.png){#fig:213 width=80%}

В результате изменения 

14. (Не у,У)далось удалить файл от имени пользователя, не являющегося его владельцем.

![](image/2_14.png){#fig:214 width=80%}

15. Повысим свои права до суперпользователя и верните атрибут t на директорию /tmp командами:

```
su -
chmod +t /tmp
exit
```

![](image/2_15.png){#fig:215 width=80%}

# Выводы по проделанной работе

## Вывод

В результате выполнения работы мы изучили механизм идентификаторов, применения SetUID- и Sticky-битов, получили практические 
навыки работы в консоли с дополнительными атрибутами, рассмотрели работы механизма смены идентификатора процессов пользователей,
а также посмотрели влияние бита Sticky на запись и удаление файлов.

Были записаны скринкасты выполнения и защиты лабораторной работы.

Ссылки на скринкасты:

- [Выполнение, Youtube]()
- [Выполнение, Rutube]()
- [Защита презентации, Youtube]()
- [Защита презентации, Rutube]()

# Список литературы

::: {#refs}
:::
