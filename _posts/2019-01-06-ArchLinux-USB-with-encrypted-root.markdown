---
layout: post
title:  "ArchLinux із шифрованою фаловою системою на USB"
subtitle: "Встановлення"
date:   2019-01-06 09:38:00
categories: [linux]
---

У цій нотатці я хочу, найперше для себе, записати послідовну інструкцію з підготовки USB "флешки" із ArchLinux та зашифрованою корневою файловою системою. Таку флешку я завжди маю при собі й за необхідності можу скористатися будь-яким (ну, майже будь-яким) ПК чи ноутом і завантажити повноцінну систему.

Це буде мінімальна істаляція без графічного середовища лише з усім необхідним особисто для мене. Конфігурацію графічного середовища я винесу в окрему нотатку, тут зосередимося лише на процесі встановлення.

Чому саме **ArchLinux**? Знаєте, дистрибутив немає значення - особисто мені на данний момент часу подобається саме Arch :)

## Перш ніж розпочати

Для встановлення на флешку, як ви вже здогадалися, нам потрібна флешка. Якщо будемо встановлювати графічне середовище (а ми будемо), офісні програми та інше, то мінімальний розмір флешки має бути **16Гб**. Іншу буде потрібно для завантаження інсталяційного образу, тут буде достатньо 4Гб (дякую, Любчик, що ненадовго позичив таку)

### Завантажувальний образ

Завантажуємо [образ][download] дистрибутиву та створюємо завантажувальний образ командою:

	dd bs=4M if=/path/archlinux.iso of=/dev/sdX status=progress && sync

де, `/path/archlinux.iso` - шлях до скачаного образу, а `/dev/sdX` - шлях до непідмонтованого USB диску.

До речі, якщо вже є встановлений ArchLinux, тоді цей крок можна пропустити і перейти відразу до встановлення.

### BIOS чи UEFI

Сьогодні існує дві систем взаємодії між материнською платою та операційною системою: устарівша [BIOS][bios] (basic input/output system) та більш сучасна [UEFI][UEFI] (unified extensible firmware interface). Оскільки я точно не знаю на якому ПК доведеться завантажувати свій Linux, мені потрібно передбачити обидва варіанта, що я й планую зробити.

### Файлова система

Як я вже писав, встановлення буду робити на зашифрований корінь. Я ж не хочу, щоб мої ключі доступів, якщо загублю флешку, потріпили у чиїсь руки? І ще кілька слів про **swap** - його не буде, оскільки я не бачу необхідності у файлі підкачи для резервної системи на флешці.

## Завантажуємо інсталяційну систему

Після того, як завантажимо систему, потрібно підключення до інтернет. Якщо у нас ідеальна ситуація із провідним інтернетом та автоматичною роздачею адресів, то все має бути просто. Перевіримо:

	ping -c1 google.com

### WiFi

Із WiFi мережею все залежить, якщо відсутні необхідні драйвера для мережевої карти, тоді ми не зможемо підключитися. Перевіримо командою:

	lspci -k | grep -A3 "Network controller"

Нічого? Тоді шукаємо кабель та провідний інтернет.

Якщо драйвер присутній, тоді глянемо на назву інтерфейсу:

	iw dev

Увімкнемо інтерфейс:

	ip link set wifiname up

І пошукаємо нашу мережу:

	iw dev wifiname scan | grep "SSID:"

Для того, щоб підключитися до мережі без шифрування, достатньо виконати команду:

	iw dev wifiname connect "networkname"

Щоб підключитися до мережі із WPA/WPA2 шифруванням потрібно потрібно виконати команду:

	wpa_supplicant -i wifiname -c <(wpa_passphrase "networkname" "password")

Якщо успішно підключилися до мережі, то тепер можемо й IP адресу отримати:

	dhcpcd wifiname

Перевіряємо:

	ping -c1 google.com

### Час

Синхронізуємо час:

	timedatectl set-ntp true


## Підготуємо флешку

Як я писав раніше, ця флешка буде сумісною з BIOS та UEFI, тому нам потрібно відповідно підготувати розмітку файлової системи. Переглянемо список пристроїв:

	lsblk

Підключаємо основну флешку і знову виконаємо цю ж команду. Бачу, що новий пристрій має назву `/dev/sdb`

### Розмітка файлової системи

Для розмітки я буду використовувати **gdisk**:

	gdisk /dev/sdb

Очистимо флешку командою **d**:

	Command (? for help): d
	No partitions

Створюємо новий GUID файлової системи:

	Command (? for help): o
	This option deletes all partitions and creates a new protective MBR.
	Proceed? (Y/N): y

Додамо 10MB MBR на початку диску:

	Command (? for help): n
	Partition number (1-128, default 1):
	First sector (34-XXXXXX), default = 64) or {+-}size{KMGTP}:
	Last sector (64-XXXXXX), default = XXXXXX) or {+-}size{KMGTP}: +10MB
	Current type is 'Linux filesystem'
	Hex code or GUID (L to show codes, Enter = 8300): EF02

Створюємо 500MB ESP:

	Command (? for help): n
	Partition number (2-128, default 2):
	First sector (34-XXXXXX), default = YYYY) or {+-}size{KMGTP}:
	Last sector (64-XXXXXX), default = XXXXXX) or {+-}size{KMGTP}: +500MB
	Current type is 'Linux filesystem'
	Hex code or GUID (L to show codes, Enter = 8300): EF00

Усе, що залишилось, форматуємо під Linux:

	Command (? for help): n
	Partition number (3-128, default 3):
	First sector (34-XXXXXX), default = YYYY) or {+-}size{KMGTP}:
	Last sector (64-XXXXXX), default = XXXXXX) or {+-}size{KMGTP}:
	Current type is 'Linux filesystem'
	Hex code or GUID (L to show codes, Enter = 8300):

Перевірю, що я натворив:

	Command (? for help): p

Має бути щось таке:

	Number  Start (sector)    End (sector)  Size       Code  Name
	   1            2048           22527   10.0 MiB    EF02  BIOS boot partition
	   2           22528         1046527   500.0 MiB   EF00  EFI System
	   3         1046528        30605278   14.1 GiB    8300  Linux filesystem

Записуємо зміни на диск:

	Command (? for help): w

### Фоматуємо файлові системи

Перевіряємо:

	lsblk /dev/sdb

Маємо побачити ось такий результат:

	NAME     MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
	sdb        8:16   1 14.6G  0 disk
	├─sdb1     8:17   1   10M  0 part
	├─sdb2     8:18   1  500M  0 part
	└─sdb3     8:19   1 14.1G  0 part

`sdb1` - не форматуємо. Це буде BIOS/MBR партиція.

Форматуємо 500MB для систем із EFI із файловою системою FAT32:

	mkfs.fat -F32 /dev/sdb2

Залишається наша корнева система, яку ми зараз зашифруємо. Назву я її `empty` - просто так цікавіше:

	cryptsetup -y -v luksFormat --type luks2 /dev/sdb3
	cryptsetup open /dev/sdb3 empty
	mkfs.ext4 /dev/mapper/empty

## Встановлюємо базову систему

Як я й говорив, буде лише необхідний мінімум пакетів.

### Монтуємо

Підключаємо корневу файлову систему:

	mount /dev/mapper/empty /mnt

Підключаємо /boot:

	mkdir /mnt/boot
	mount /dev/sdb2 /mnt/boot

### Базова система

Встановлюємо базові пакети:

	pacstrap /mnt base base-devel

### fstab

Створюємо файл із партиціями:

	genfstab -U /mnt >> /mnt/etc/fstab

## Базова конфігурація системи

Оскільки я не знаю на якому залізі буду завантажувати флешку, спробую передбачити максимальну кількість необхідних пакетів та драйверів.

### PARTUUID

Проте, перш ніж змінити корінь, нам потрібно дізнатися ідентифікатор шифрованої партиції. Виконуємо команду:

	blkid

Маємо побачити щось таке:

	/dev/sdb1: PARTLABEL="BIOS boot partition" PARTUUID="9c57888f-b4df-4a9c-9180-61df16492a91"
	/dev/sdb2: UUID="F3B4-203F" TYPE="vfat" PARTLABEL="EFI System" PARTUUID="5282a0c4-0a17-4f19-95d4-cba48ec75773"
	/dev/sdb3: UUID="9a94e427-e405-44d9-8ce8-75a30c511a80" TYPE="crypto_LUKS" PARTLABEL="Linux filesystem" PARTUUID="7369da28-d876-4f9b-9b57-6104d0d2ef79"
	/dev/mapper/empty: UUID="7623ee46-74f1-4f2a-b5f0-30dfa11be3f4" TYPE="ext4"

Копіюємо ідентифікатор `sdb3`, а саме:

	PARTUUID="7369da28-d876-4f9b-9b57-6104d0d2ef79"

Він нам знадобиться пізнаше для налаштування завантажувача.

### chroot

Заходимо у нове середовище:

	arch-chroot /mnt /bin/bash

### Часова зона

Видаляємо поточне налаштування:

	rm /etc/localtime

Підключаю Київ:

    ln -s /usr/share/zoneinfo/Europe/Kiev /etc/localtime

Створюємо /etc/adjtime:

	hwclock --systohc

Дальше мені потрібно обрати локаль:

	vi /etc/locale.gen

Я обрав `en_US.UTF-8 UTF-8` та `uk_UA.UTF-8 UTF-8`

Генеруємо локалізації:

	locale-gen

Задаємо змінну `LANG` у файлі `/etc/locale.conf`

	echo LANG=uk_UA.UTF-8 > /etc/locale.conf

### hostname

Створимо файл /etc/hostname:

	echo usb > /etc/hostname

Редагую /etc/hosts:

	vi /etc/hosts

Додам рядок:

	127.0.1.1    usb.localdomain    usb

### Образ ядра

Для того, щоб ядро завантажилося коректно і ми могли підмонтувати зашифровану файлову систему, нам потрібно внести правки у файл `mkinitcpio.conf`:

	vi /etc/mkinitcpio.conf

Наш рядок із `HOOKS` має виглядати ось так:

	HOOKS=(base udev autodetect modconf block encrypt filesystems keyboard fsck)

Генеруємо RAM образ:

	mkinitcpio -p linux

### Журнал логів

Щоб не перевантажувати нашу флешку непотрібними логами, мінімалізуємо їх розмір. У файлі `/etc/systemd/journald.conf` необхідно змінити два параметри:

	Storage=volatile
	SystemMaxUse=16M

### Завантажувач

Встановлюємо grub та efibootmgr пакети:

	pacman -S grub efibootmgr

Вносимо правки у файлі `/etc/default/grub` для коректного підключення шифрованого кореневого тому (ось тепер і використаємо `PARTUUID`, що запам'ятовували вище):

	GRUB_DEFAULT=0
	GRUB_TIMEOUT=2
	GRUB_DISTRIBUTOR="Encrypted"
	GRUB_CMDLINE_LINUX_DEFAULT="quiet"
	GRUB_CMDLINE_LINUX="cryptdevice=PARTUUID=7369da28-d876-4f9b-9b57-6104d0d2ef79:empty"
	GRUB_PRELOAD_MODULES="part_gpt part_msdos"

Тепер можемо встановити для режиму MBR/BIOS:

	grub-install --target=i386-pc --boot-directory /boot /dev/sdb

Та для UEFI:

	grub-install --target=x86_64-efi --efi-directory /boot --boot-directory /boot --removable

Генеруємо конфігураційний файл GRUB:

	grub-mkconfig -o /boot/grub/grub.cfg

### Підтримка мережі

Особисто я надаю перевагу [NetworkManager][NetworkManager], но, поки ми будемо без графічного інтерфейсу, нам потрібно щось простіше - управління мережею із консолі. Давайте встановимо необхідні пакети:

	pacman -S ifplugd iw wpa_supplicant dialog

Щоб точно бути впевненим у назвах мережевих інтерфейсів `eth0` та `wlan0`, виконаємо команду:

	ln -s /dev/null /etc/udev/rules.d/80-net-setup-link.rules


### Відео драйвера

Для підтримки більшості популярних відеокарт необхідно встановити:

	pacman -S xf86-video-ati xf86-video-intel xf86-video-nouveau xf86-video-vesa

### Підтримка touchpad

Як же без touchpad на ноуткубах:

	pacman -S xf86-input-synaptics

### Батарея

Ну, ви зрозуміли:

	pacman -S acpi

## Користувач нової системи

Найперше задамо пароль для користувача **root**

	passwd

### Користувач з правами sudo

Створюю користувача для себе із необхідними доступами:

    useradd -m -g users -G audio,disk,lp,network,power,storage,video,wheel -s /bin/bash kovalyshyn

Якщо я не хочу вводити пароль кожного разу при виклику команди sudo, тоді у файлі `/etc/sudoers` відкриваємо рядок:

	%wheel ALL=(ALL) NOPASSWD: ALL

І додаємо ще один пакет

	pacman -S polkit

## Нова система

Виходимо із режиму зміни кореневої системи:

	exit

Відключаємо диски та завантажуємося із нашої нової флешки:

	umount /mnt/boot /mnt
	reboot

Перед завантаженням ви маєте ввести пароль до файлової системи. Якщо пароль правильний, тоді система завантажиться.

Оскільки ми ще не встановили жодного графічного середовища, то заходимо у консоль Linux

### Провідна мережа

Запускаємо службу, що відповідає за автоматичне налаштування провідної мережі:

	cp /etc/netctl/examples/ethernet-dhcp /etc/netctl/eth0-arch_usb
	systemctl start netctl-ifplugd@eth0.service


### Wifi

Для підключення до Wifi мережі використовуємо команду:

	wifi-menu -o

### Час

Корегуємо час через інтернет протокол:

	timedatectl set-ntp true

## Фінал

Усе! Наша флешка готова! Залишилось налаштувати графічне середовище. Але це вже [материал для іншої нотатки]({{ site.baseurl }}{% post_url 2019-01-12-ArchLinux-USB-i3 %}).


[download]: https://www.archlinux.org/download/
[BIOS]: https://en.wikipedia.org/wiki/BIOS
[UEFI]: https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface
[NetworkManager]: https://wiki.archlinux.org/index.php/NetworkManager
