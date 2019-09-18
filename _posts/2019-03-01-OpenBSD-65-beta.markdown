---
layout: post
title: "OpenBSD 6.5 beta"
subtitle: "А ось і перший досвід з бетками 😁"
date: 2019-03-01 18:52:00
categories: [bsd]
---

Ще не завершив нотатку про [встановлення та налаштування OpenBSD]({{ site.baseurl }}{% post_url 2019-02-23-encrypted-OpenBSD-on-ThinkPad-X240 %}), як вчора стала доступною beta версії 6.5. А чому би не обновити свій `ThinkPad X240` до 6.5? Давай оновляти!


# Оновлення

Я вирішив, що оновлюватиму з допомогою USB флешки, так, як і встановлював. Завантажую образ:

    ftp https://cloudflare.cdn.openbsd.org/pub/OpenBSD/snapshots/amd64/install65.fs

Та записую на USB диск:

    dd if=install65.fs of=/dev/rsd2c bs=1m

Перевантаження, обрати завантаження з USB та й вперед!

## Повне шифрування диску

Оскільки мій диск зашифровано, то потрібно обрати пункт `(S)hell` та відкрити наш диск.

    bioctl -c C -l sd0a softraid0

Після введення паролю, ви маєте побачити напис:

    CRYPTO volume attached as sd2.

Тепер можемо спокійно виходити із консолі `exit` та обирати `(U)pgrade`. Процес доволі простий та не займає багато часу. Після завершення - перевантажую в оновлену систему.

У новій системі оновлюю всі встановлені програми:

    pkg_add -u
    sysmerge -d

# OpenBSD 6.5

    OpenBSD 6.5-beta (GENERIC) #727: Thu Feb 28 00:11:52 MST 2019
    deraadt@amd64.openbsd.org:/usr/src/sys/arch/amd64/compile/GENERIC
