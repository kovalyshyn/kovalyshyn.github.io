---
layout: post
title: "Підключаю LTE модем під OpenBSD"
subtitle: "Настав час перевірити модем на ThinkPad X240"
date: 2019-03-30 20:52:00
categories: [bsd]
---

Мій `ThinkPad X240` оснащено LTE модемо. Проте, я жодного разу не пробував його запустити під `ArchLinix`. А ось під `OpenBSD` вирішив спробувати, тимпаче, що у мене є незадіяна LTE карточка Kyivstar. Я довго шукав в інтернетах, як таке зробити, але нічого конкретного. А потім я натрапив на документацію [umb(4)][umb], де знайшов приклади та інформацію, що [мій модем підтримується][umsm]. Залишилось вияснити як саме його задіяти. Виявляється, усе банально просто!


Прописуємо APN:

    doas ifconfig umb0 apn internet

Під'єднуємось до мережі:

    doas ifconfig umb0 up

Я побачив, що з'єднання встановилося, проте інтернету не було. Як виявилося, необхідно додатково ще прописати самостійно DNS та route. Також, я вирішив перевірити швидкість з'єднання для усіх операторів.

## Kyivstar

![Kyivstar](/assets/posts/lte-kyivstar.png)

## Lifecell

![Lifecell](/assets/posts/lte-lifecell.png)

## Vodafon

![Vodafon](/assets/posts/lte-vodafon.png)


[umb]: https://man.openbsd.org/umb.4
[umsm]: https://man.openbsd.org/umsm.4
