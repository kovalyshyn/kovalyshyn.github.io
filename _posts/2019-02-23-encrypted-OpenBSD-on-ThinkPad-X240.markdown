---
layout: post
title: "Встановлюю OpenBSD на ThinkPad X240 з шифруванням файлової системи"
subtitle: "Власний досвід безпечного встановлення системи на X240 😜"
date: 2019-02-23 17:14:14
categories: [bsd]
---

Я вже [розповідав]({{ site.baseurl }}{% post_url 2019-02-09-OpenBSD-on-a-laptop %}), що здійснив перехід на `OpenBSD`. Я й надалі використовую [Arch Linux]({{ site.baseurl }}{% post_url 2019-01-06-ArchLinux-USB-with-encrypted-root %}) на роботі (усе через специфічне ПЗ), але власний ноут тепер тішить мене BSD системою.

Якщо ви не знаєте, то BSD системи (а їх є кілька сьогодні), на відміну від Linux, є повноцінними операційними системами, а не лише ядром + GNU утиліти. Тож моя ціль була встановити мінімальну систему, максимально використовувати базові компоненти та... Приєднатись до [темної сторони сили][OpenBSD]. Я знаю, що більшість розглядає `BSD`, як чудовий маршрутизатор чи поштовий сервер, а я спробую розглянути як операційну систему для щоденних задач на `ThinkPad X240`.

# Встановлення

Не злукавлю, якщо скажу, що в OpenBSD один із найпростіших процесів встановлення з усього сімейства BSD систем. Просто завантажте образ:

    curl -OJ https://cdn.openbsd.org/pub/OpenBSD/6.4/amd64/install64.fs

Та запишіть на USB диск:

    dd if=install64.fs of=/dev/sdb bs=1m

Оберіть завантаження з USB та й вперед!

## Повне шифрування диску

[Моя](Моя) ціль - бути певним, що навіть якщо я втрачу цей ноут, то мої данні будуть недоступні. Щоб встановити систему на повністю зашифрований диск, необхідно обрати пункт `(S)hell` та створити програмний RAID з шифруванням.

Створюємо новий EFI розділ:

    fdisk -iy -g -b 960 sd0

А тепер можемо й [RAID][softraid] організувати:

    disklabel -E sd0
    Label editor (enter '?' for help at any prompt)
    > a a
    offset: [1024]
    size: [500117105]
    FS type: [4.2BSD] RAID
    > w
    > q
    No label changes.

Переходимо до шифрування нашого програмного RAID:

    bioctl -c C -l sd0a softraid0

Після введення паролю, ви маєте побачити напис:

    CRYPTO volume attached as sd2.

Тепер можемо спокійно виходити із консолі `exit` та обирати `(I)nstall`.

## OpenBSD

Інсталятор задав мені кілька запитань: розкладка, назва хоста, пароль для root та інше. Можливо, система не розпізнає вашу wifi карту, тому краще встановлювати через звичайну провідну мережу, що я й робив.

Оскільки я встановлюю це все на ноутбук, то мені буде потрібно графічне середовище - не забуваємо активувати [xenodm][xenodm] та відключити старт `sshd`.

Переходимо до розмітки диску. Ось тут важливо обрати диск, який було зашифровано та підключено у попередньому пункті:

   Which disk is the root disk ('?' for details) [sd0] sd2

Я не буду розповідати як саме розмітив свій диск, скажу лише так: якщо у вас більше 120Гб та ви не знаєте, як краще розмітити диск - автоматичний розподіл вам підійде!

Перевантажуємо систему та завантажуємо OpenBSD

# Налаштування мережі

Після входи я потрапив у графічне середовище [fvwm][fvwm]:

![fvwm](https://www.gabsoftware.com/wp-content/uploads/2011/04/Capture-12.png)
Я до цього повернуся пізніше, найперше потрібно налаштувати wifi, щоб не бути прив'язаним.

Відкриваємо [xterm][xterm], переходимо під root з командою [su][su] та завантажуємо необхідні драйвера:

    fw_update

Команда [fw_update][fw_update] завантажить та встановить необхідні драйвера.

Одна з найпрекрасніших речей в OpenBSD - більшість речей інтегрована у систему і немає необхідності встановлювати додаткові утиліти для налаштування мережі чи підключення до WPA2 мереж. Вмикаємо мережку та скануємо:

    ifconfig iwn0 up
    ifconfig iwn0 scan

Коли побачив свою мережу - просто створюю [файл із назвою][hostname.if] мережевого адаптера вкінці `/etc/hostname.iwn0` та прописую параметри підключення:


    join "YOUR_SSID" wpakey "YOUR_PASSPHRASE"
    # you can specify other networks here too, in order of priority:
    # join "WORK_SSID" wpakey "WORK_PASSPHRASE"
    # join "OPEN_COFFEE_SHOP"
    dhcp
    inet6 autoconf
    up powersave

Ось і все! Тепер мій ноут автоматично буде підключатися до тієї мережі, в якої сильніше сигнал. Щоб не перевантажувати систему, достатньо перевантажити лише [мережу][netstart]:

    ifconfig em0 down
    ifconfig iwn0 down
    pkill dhclient
    sh /etc/netstart

# Базове налаштування

Найперше, що я зробив - відключив набридливе вікно з [xconsole][xconsole]:

    sed -i 's/xconsole/#xconsole/' /etc/X11/xenodm/Xsetup_0
    echo 'xset b off' >> /etc/X11/xenodm/Xsetup_0

Також, мені не подобається `beep` звук логіну. Для цього необхідно додати у файл `/etc/wsconsctl.conf`:

    keyboard.bell.volume=0

OpenBSD немає `sudo`, проте є [doas][doas]:

    echo 'permit persist keepenv YOUR_USERNAME' > /etc/doas.conf

Оскільки це ноут, то вартує увімкнути power manager. Пропишу автоматичну адаптацію CPU, а також, якщо залишатиметься 6% - відправляти ноут в ґібернацію:

    rcctl enable apmd
    rcctl set apmd flags -A -Z 6
    rcctl start apmd

Потрібно подякувати розробникам, оскільки на `ThinkPad` сон та ґібернація просто працюють. Вам нічого не потрібно налаштовувати. Єдина річ, що я додаю, це щоб після того, як закриваю кришку ноута, автоматично блокувався екран. У файл `/etc/apm/suspend` додаю вміст:

    #!/bin/sh
    pkill -USR1 xidle

Та робимо executeble:

    chmod +x /etc/apm/suspend

Додаю власного користувача в групу `staff`:

    usermod -G staff YOUR_USERNAME

В файлі `/etc/login.conf` прописую параметри:

    staff:\
      :datasize-cur=2048M:\
      :datasize-max=4096M:\
      :maxproc-cur=512:\
      :maxproc-max=1024:\
      :openfiles-cur=4096:\
      :openfiles-max=8192:\
      :stacksize-cur=32M:\
      :ignorenologin:\
      :requirehome@:\
      :tc=default:

І трішки параметрів ядра з допомогою [sysctls][sysctls], у файлі `/etc/sysctl.conf` пропишу:

    # shared memory limits (chrome needs a ton)
    kern.shminfo.shmall=3145728
    kern.shminfo.shmmax=2147483647
    kern.shminfo.shmmni=1024

    # semaphores
    kern.shminfo.shmseg=1024
    kern.seminfo.semmns=4096
    kern.seminfo.semmni=1024

    kern.maxproc=32768
    kern.maxfiles=65535
    kern.bufcachepercent=90
    kern.maxvnodes=262144
    kern.somaxconn=2048

    # record audio
    kern.audio.record=1

    # refer to xf86(4) for details
    machdep.allowaperture=1

Щоб подовжити життя своєму SSD, включаємо [softupdates][softupdates] у файлі `/etc/fstab`:

    71bb88c2ddd0128c.b none swap sw
    71bb88c2ddd0128c.a / ffs rw,softdep,noatime 1 1
    71bb88c2ddd0128c.e /home ffs rw,nodev,nosuid,softdep,noatime 1 2
    71bb88c2ddd0128c.d /usr ffs rw,wxallowed,softdep,noatime,nodev 1 2

# Налаштовую X11

Як я писав раніше, за замовчуванням OpenBSD пропонує `fvwm`. Особисто для мене це середовище виявилося не зручним. Звичайно, була ідея зібрати [dwm]({{ site.baseurl }}{% post_url 2019-01-26-suckless-software %}), яким я користуюсь на `Arch`. Проте, спробую притримуватися основної ідеї: максимально базовий функціонал. І тут я дізнався про наявність [cwm][cwm]. У цій нотатці я лише скажу, що все ще вивчаю та порівнюю. А що з цього вийшло - думаю, що напишу вже іншим разом.

# Підсумок

Я встановив [OpenBSD][OpenBSD] на своєму `ThinkPad X240`. Користуватимусь щоденно і побачимо, що з цього вийде.


[OpenBSD]: https://www.openbsd.org/
[softraid]: https://man.openbsd.org/softraid.4
[xenodm]: https://man.openbsd.org/xenodm.1
[fvwm]: https://man.openbsd.org/fvwm
[xterm]: https://man.openbsd.org/xterm.1
[su]: https://man.openbsd.org/su.1
[fw_update]: https://man.openbsd.org/fw_update
[hostname.if]: https://man.openbsd.org/hostname.if.5
[netstart]: https://man.openbsd.org/netstart
[xconsole]: https://man.openbsd.org/xconsole.1
[doas]: https://man.openbsd.org/OpenBSD-6.0/doas.1
[sysctls]: https://man.openbsd.org/sysctl.2
[softupdates]: https://www.openbsd.org/faq/faq14.html#SoftUpdates
[cwm]: https://man.openbsd.org/cwm.1
