---
layout: post
title:  "ArchLinux на USB з віконним менеджером i3"
subtitle: "Налаштування"
date:   2019-01-12 19:11:00
categories: [linux]
---

Продовжуючи [попередній запис]({{ site.baseurl }}{% post_url 2019-01-06-ArchLinux-USB-with-encrypted-root %}), сьогодні опишу процес встановлення графічного середовища для нашої USB "флешки". Звичайно, ви можете встановити XFCE, що має доволі непогано розміститися на 16Гб, проте я надам перевагу віконному менеджеру [i3][i3], а зокрема його форк - [i3-gaps][i3-gaps].


## Встановлюємо необхідні пакети

Я підготував список базових пакетів, які встановлені у мене. Це віконний менеджер, офісний пакет та шрифти:

    sudo pacman -Syu curl
    curl https://raw.githubusercontent.com/kovalyshyn/archrice/master/packages/main | sudo pacman -S -

## AUR

Після встановлення, я хочу підключити репозиторій користувачів [AUR][AUR], для цього мені потрібно зібрати пакет `yay`:

    cd /tmp
    git clone https://aur.archlinux.org/yay.git
    cd yay
    makepkg -si

Після встановлення `yay` я можу встановити додаткові шрифти та програми:

    yay -Syu ttf-emojione ttf-font-awesome ttf-ms-fonts ttf-symbola ttf-windows unclutter-xfixes-git xkblayout-state-git xrectsel ncpamixer gotop-bin vim-gruvbox-git

## Конфігураційні файли та графічне середовище

І так, у мене вже встановлено все необхідне, потрібно підготувати графічне середовище. Для цього я скористаюся зі своїх готових конфігураційних файлів:

    git clone https://github.com/kovalyshyn/archrice.git
    cd archrice
    cp -rf .b* ˜/
    cp -rf .c* ˜/
    cp -rf .i* ˜/
    cp -rf .p* ˜/
    cp -rf .v* ˜/
    cp -rf .x* ˜/
    cp -rf .X* ˜/
    curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

У конфігураційному файлі [.xinitrc][.xinitrc] прописано запуск **i3**, якщо я зараз просто напишу в консолі `startx` то відбудеться завантаження графічного середовища. Я не використовую жодного графічного менеджера для входу користувача, а надаю перевагу самостійному запуску. Проте, щоб кожного разу не писати `startx`, я додав у файлі [.profile][.profile], що якщо реєстрація на першому терміналі - відразу запускати `X`

Встановлюємо термінал. Мій вибір - [st][st] із додатковими патчами:

    git clone https://github.com/LukeSmithxyz/st.git
    make
    sudo make install


## Результат

Тепер я можу спокійно вийти і повторно зайти, щоб запустилося моє типове графічне середовище з **i3**:

![my ArchLinux i3](http://pub.webitel.com/_/Ic7SRCQfabVOD5VMgYv3LzGliN3nIP.png)

Якщо вам цікаво, що у мене налаштовано та як воно працює - пишіть у коментарях, я підготую детальний опис, чи, можливо буде краще, запишу відео.


[i3]: https://wiki.archlinux.org/index.php/i3
[i3-gaps]: https://github.com/Airblader/i3
[AUR]: https://wiki.archlinux.org/index.php/Arch_User_Repository
[.xinitrc]: https://github.com/kovalyshyn/archrice/blob/master/.xinitrc
[.profile]: https://github.com/kovalyshyn/archrice/blob/master/.profile#L26
[st]: https://st.suckless.org/