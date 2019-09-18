---
layout: post
title: "Робота веб-камери в OpenBSD"
subtitle: "Нарешті мені вдалося запустити свою web-камеру"
date: 2019-02-22 16:58:00
categories: [bsd]
---

Підходить до завершення другий тиждень роботи з `OpenBSD`. Більшість вже нелаштував, проте, ніяк не вдавалося запустити web-камеру. Усі пишуть, що працює, а в мене - ні. Як виявилося проблема з [USB3.0 та OpenBSD6.4][openbsd mail list] - камера не працює. В BIOS відключив підтримку USB3.0 і, о чудо, все "просто запрацювало" без танців з бубнами:

![OpenBSD video](/assets/posts/2019-02-22_18-51-00_screenshot.png)

PS: камеру запускаю командою:

    mpv --geometry=-0-0 --autofit=25%  av://v4l2:/dev/video0

[openbsd mail list]: https://marc.info/?l=openbsd-misc&m=155057838803889&w=2
