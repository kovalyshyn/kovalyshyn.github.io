---
layout: post
title: "Як роздрукувати текст з vim редактора?"
subtitle: "Або з neovim редактора - без різниці"
date: 2019-09-16 19:37:00
categories: [linux, bsd]
---

Вже доволі давно основним редактором для мене є `neovim`. Проте, лише сьогодні у мене виникла необхідність роздрукувати набраний текст і я не знав, як це правильно зробити... Після певного часу пошуку та читання, я зрозумів, що можу експортувати текст у [PostScript][ps], а вже з нього роздрукувати на принтері:

    :hardcopy > test.ps

Та пробую відкрити файл з допомогою [zathura][zathura] - порожній файл... Ага, не встановлено розширення для PostScript - `zathura-ps`. Встановив. Відкриваю та бачу ось таке:

![wrong encoding](/assets/posts/vim-print-encoding.png)

Почав шукати далі й виявилося, що для екпорту в ps мені потрібно вказати 8-бітне кодування, а не utf8. Тому я додав у конфігураційний файл `nvim` наступний рядок:

    set printencoding=koi8-u

Та спробував знову зробити експорт:

![ok encoding](/assets/posts/vim-print-ok.png)

Чудово! А тепер ще додам у свій конфіг, щоб відразу робити експорт та відкривати файл у `zathura` одним натисканням кнопки `F6`:

    map <F6> <Esc>:w!<CR><Esc>:hardcopy >/tmp/p.ps<CR><Esc>:!zathura /tmp/p.ps&<CR><Esc>

Готово!

[ps]: https://neovim.io/doc/user/print.html
[zathura]: https://github.com/pwmt/zathura
