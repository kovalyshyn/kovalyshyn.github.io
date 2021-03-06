---
layout: post
title:  "XTerm не такий уже й поганий 🤨"
subtitle: "У пошуках ідеального терміналу"
date:   2019-02-01 19:41:32
categories: [linux]
---

На початку року я розпочав перехід на [suckless]({{ site.baseurl }}{% post_url 2019-01-26-suckless-software %}) програмне забезпечення, а сьогодні зробив ще одне цікаве відкриття - `XTerm`! Для мене цей термінал завжди асоціювався із чимось жахливим (ще з часів FreeBSD на початку 2k років): некрасиві шрифти без підтримки UTF8, "не знаю як тут налаштувати тему", відсутність нормальної можливості копіювати текст чи просто його виділити. Ось так він виглядає за замовчуванням:

![default XTerm](http://pub.webitel.com/_/GzdaaBDPRhD5lctI567gSBAf9YSJ9S.png)

Я пробував різні термінали. Останній, на якому я зупинився, був [st][st]. Із набором необхідних патчів він дуже і дуже 😘. Але я ніяк не міг налаштувати перегляд картинок у терміналі. Не працює і все тут!

Я знову відновив пошук "ідеального" терміналу. Однак одні мені не зайшли, через розмір бінарників та кількість непотрібних фіч. Інші - не вистачало потрібного функціоналу. Тут я згадав про XTerm й подумав: "якщо з `st` зміг розібратися, то може й цей не такий поганий?". І знаєте що? Він крутий! Усе необхідне конфігуриться через `.Xdefaults` файл. Налаштовуєш шрифти, виділення, копіювання, прокрутку - усе, чого так не вистачало. Мій поточний файл [.Xdefaults][.Xdefaults]. А ось, який
тепер вигляд має мій `XTerm`:

![my XTerm](http://pub.webitel.com/_/OryLSzBjY9hlBLxOULSSyANTCses6P.png)

А головне - перегляд картинок у терміналі працює! Я тут відео навіть записав на [youtube][youtube]. Тепер це мій основний термінал 🥰.

[st]: http://st.suckless.org/
[.Xdefaults]: https://github.com/kovalyshyn/home/blob/master/.Xdefaults
[youtube]: https://www.youtube.com/watch?v=YNcW8AbnFLw
