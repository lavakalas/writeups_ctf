# Пентест

## 01. Отправная точка

*Александр Кот* и какие-то *рубероиды*. Гуглим что такое рубероиды, и зачем кому-то их цена

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/2181d382-1cb4-4d83-8073-32e8a1f3b7e5)

Получаем *Одессу*

Ищем в ВК Александр Кот в городе Одесса, находим сию [страницу](https://vk.com/hstmst?from=search) и первый флаг

**Ответ:** H0CTF{Th1S_1S_Th3_ST4R7_p01N7}

## 02. Ого, вебсайт

Листаем страницу нашего Кота, замечаем ссыль

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/8cc7d1a4-6db4-495a-8adb-e1c69213ccb5)

Линк оказывается битый

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/23890adf-3987-4f58-8f30-34ab977c2cce)

Однако ж "**он** помнит всё"

Запускаем [Wayback Machine](https://web.archive.org/web/20240501000000*/https://ibb.co/pr1NJcM), есть снэпшот от 5 января

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/e70d0028-08d0-4034-afae-4140cb21a273)

Гуглим `mojang monster Восхитительный сервис! Быстрое соединение`

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/3ef33cfb-d75a-4818-b9a7-dd920073bb03)

Первый же линк donthackme.ru

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/9e6ddb73-ed44-4783-92fa-2622aa1b884a)

**Ответ:** H0CTF{D0RKs_D0_R1gh7_Th1NGs}

## 03. Кто это написал?

Пронюхиваемся по сайту, ловим пару рикроллов, видим комментарии. Начинаем настоящий пентестъ.
В фоне запускаем nmap, наверняка пригодится

Открываем миру F12, заходим в Network, перезагружаем пейдж

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/e7593832-fc49-468b-8c72-8c086daf56b3)

Видим некую getcomments пыху, глянем поближе

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/63873ff4-0b81-4a42-a9c6-7407420a2eaa)

В адрексной строке переданы параметры **per_page** и **page**, их же нам возвращает и пыха в JSON. В той же джсонке есть ещё параметры total_pages, который меняется в зависимости от пар-а per_page, и **approved**, который нам теперь и интересен.
По дефолту он имеет значение true, пробуем передать в него false.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/88e57f31-9c2c-462c-acf0-61f3e0818728)

И вот они, гневыне комментарии бедняжек.
[FazaN, обязательно глянь.](https://shorturl.at/LFov1)
Страниц много, попробуем выгрузить питоном.

```
import requests, pprint, json

for page in range(1, 37):
    r = requests.get(f"https://donthackme.ru/api/getcomments.php?per_page=15&page={page}&approved=false")
    pprint.pprint(json.loads(r.text))
```

`python parse.py >> comments.txt`

Среди обещаний находим третий флаг.

**Ответ:** H0CTF{4P1_M4y_b3_Uns4F3_T0O}

## 04. Безобидный файлик

Отдавший нам третий флаг JVX_HACKER обещал выложить WiFi трафик. И выложил в телеге.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/4e3212f7-2fcf-4069-a141-501385b15c75)

Ну хоть Russian с большой буквы написал, горжусь. Файл пожат и назан sniffed.tar.gz. Его название и есть четвёртый флаг.

**Ответ: sniffed.tar.gz**

## 06. Не меняйте пароли

Ранее запущенный nmap нашёл robots.txt и сабдомен wmail79.donthackme.ru, переходим.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/fe1564e2-e6fe-4cbe-9e2f-9b5096783fc3)

Ага, у нас есть форма логина на сабдомене. Чтож БОЛЬШЕ ФАЗЗА БОГУ ФАЗЗА

`ffuf -c -w /usr/share/wordlists/dirb/big.txt -u "https://wmail79.donthackme.ru/FUZZ"  -H "User-Agent: " -e ".php,.txt,.html,.js,.conf"`

НаФАЗЗили вкусное

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/d7956f92-6b11-4309-8673-7b5004a8a73c)

Смотрим https://wmail79.donthackme.ru/inbox.php Редиректит обратно на форму логина.
А если попробовать бурпой?

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/3cd660c8-79c7-4ecf-b0f2-0a87b65720ee)

Нашли сладенькое. curl тоже вернёт страницу ящика ибо не идёт по ридеректу по дефолту.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/a197e1e9-ef6f-4e4f-942a-c67b43e343ef)

Видна попытка послать запрос на `/viewEML.php?msg=0 FLAG&folder=inbox`

curl и тут нам поможет.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/6dd0fa33-d1fb-4458-b0d3-0aee08cbd22a)

**Ответ:** H0CTF{L34KeD_PWDs_Op3n_D00rS}

## 07. Где-то в дампах

Среди входящих сообщений находим следующее

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/cc0bf9a4-9a6d-4523-869e-d3149213d2b6)

В приложении есть некий log.txt, при попытке открыть которого делается след запрос

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/e7a86970-955d-4e7e-ac30-37e44a3fa920)

вставляем линк к почтовому домену https://wmail79.donthackme.ru/nMf293uhLfl2hi/86%20System%20Message.eml/log.txt

Делаем поиск по "/"

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/fae03a42-7ab7-45d4-98ec-b5fd729af27f)

Топаем обратно на основной домен и находим дамп БДшки

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/f85f3ab8-59a7-4f91-84b6-4bb45c7c2baa)

**Ответ:** H0CTF{D0_YoU_R34d_L0gs_HUH}

## 05. Сквозь тернии к...

Рассмотрим sniffed.cap поближе. aircrack-ng рассмотрел в нём несколько точек, от одной есть хендшейк. Брутим пароль 8-ю цифрами

Получаем пароль от WiFi == 11031943

Расшифровываем траффик сетки в Wireshark

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/e0f58957-2044-46bc-88d6-16e9863c3774)

В отдельных запросах находим мыло darkestpart@gmail.com и на breachdirectory пароль от него -- sources00, но они нам уже не нужны, ибо мы обошли логин почты.

Интересно нам сейчас расположение админки -- /admin_f7Z0pjDe3LmeR1/login.php

Переходим -- ![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/81b8997e-49d7-4174-9c81-134da258d161)

Пробуем обойти IP фильтр. Самая распространённая техника обхода -- заголовок X-Forwarded-For: <SPOOFED_IP>

В украденом трафике видим респонс от info.php со строкой "Ваш IP адресс: 185.193.196.99"
![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/014eb7d8-dc7b-4de2-8cba-497e79596247)

Пробуем подставить заголовок, я для этого воспользовался прокси от бёрп сьюта
![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/68a4af54-ae27-4062-a5b9-e814e9e82266)

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/2a308124-e425-43a3-bc0e-6a318bd97a81)

**Ответ:** H0CTF{X_F0Rw4Rd3d_FOR_Byp4sS}

## 08. Вапросав многа

Теперь задача -- зайти в админку. У нас есть логин и хэш пароля с известной солью из дампа БД, добытого с почты->логов. Посмотрим на хэш поближе

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/51e02906-acf8-4017-8d10-bf313b991240)

Хашид подсказывает, что вероятный формат -- md5 с солью

Глянем форматы котёнка

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/1ec264cb-6e8d-4b54-b6b5-ae7a9f43982d)

Форматы 10, 20 и 3800 не подошли, но звёзды сошлись на формате m3710

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/0a98495a-cd18-43cd-aa17-2f704970f8d8)

Логинимся в админку и отвечаем на контрольные вопросы, пользуясь собранной для I-III инфой.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/345e4846-3def-4b9a-b6ed-917f5dcfb15f)

**Ответ:** H0CTF{T00_S7roNG_2FA_0r_NoT}

## 09. Защита от защиты

upload.php требует пароль, печально. Но не сдаёмся! Начинаем насиловать comments.php. В этот раз он функционирует иначе... Теперь при передаче билиберды в ?approved= сервер возвращает 500, а не устанавливает в approved дефолтные значения. Рассмотрим поближе, а именно с точки зрения SQL инъекций.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/67e263c7-1030-4b91-b013-ee62ca583cf1)

Получилось дважды селектнуть из одной бдшки, теперь пробуем другие таблицы

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/26af5a97-c099-4ef0-96b4-6565eda07f99)

Ура! Вещи похожи на b64_encoded, поэтому бежим в кибершеф

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/6a51da67-2814-48a7-810d-9a14cc1959d6)

а ключ не декодится, пока что оставием его как есть

хмм, некий iv и ключ со странной строкой, гуглим IV cipher, понимаем, что шифр -- aes

успешно декодим и получаем пароль от upload.php

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/fb3f7eec-265c-4fa6-903f-f90a24a9cd06)

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/b26ddee9-47a2-4d56-aeb9-a76e6c81e4dc)

**Ответ:** H0CTF{SQL1_4nD_3NcRyP710N}
## 10. Заметь слона

Пытаемся залить php-shell, ловит по расширению. А если .php3? Ловит по внутренностям, однако пропускает, если всё чисто.

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/16156a94-7b6d-4683-8cbd-dd1eb263de87)


При попытке загрузить файл сильно больше, машинка подтупливает с ответом, а если расположить php-код под отвлекающей шапкой?

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/ca80fe9d-ad8f-400b-84e4-091f39fb8d4a)

Юху! Мы на*брали баллов!

Ранее в ФАЗЗЕ был замечен фалйики https://donthackme.ru/api/getfile.php, пробуем подставить новоиспечённый fileid

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/40c676ba-ad5d-43e8-b5ad-296e0cf4a0fc)

***I'M IN***
![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/d143ce05-4a25-4681-b100-f9e11abbde13)

далее классика ползания по линуксу

`ls -la .`
`cat 0000f4k_Fl4G_H3re_f4k0000z3D.txt`

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/9b8511bb-54c5-4c0d-9fed-3884a7f04a82)

**Ответ:** H0CTF{Upl04D_R3S7R1cti0Ns_BYp4SS}

## 11. Мы дома... У кого?

Для удобства дальнейшего проникновения прокидываем ревёрс шелл себе на C&C сервер

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/6da72f36-fbdf-4df3-9493-54776e204a3e) (стандартный ревёрс шелл через openbsd-netcat)

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/1700c1f4-8d9b-46ee-81b4-6bd63a7dbaf3)

тк пароля от http мы не знаем, начинаем искать второй самый страшный сон сисадмина -- SUID

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/b9498486-9c5a-4157-ad35-aaa191848aa9)

Нашли кастомный файлик exec_srvstate. Подберёмся к нему поближе. А рядом-то лежит исходник

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/a4039089-a925-4c50-8603-753af937c212)

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/6cef7bc7-2130-4394-8000-8862bbfe25f7)

Расследуем далее

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/1a37b65d-82c2-4a91-9224-53ecda3ed1ec)

Ууууу, да у нас тут eval(), выстреел в ногу и рикошет в достоинство. А файл /usr/share/nginx/html/state ещё и редактировать от http можно.

Ну чтож поодкинем туда __impoort__('os').system('/bin/sh')

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/b47a6a0d-9c16-4865-a348-1540ecaf5ed7)

Лезем в дом

![image](https://github.com/lavakalas/writeups_ctf/assets/42173474/f2c6f6f8-8195-475c-bf15-c868b748cf90)

**Ответ:** H0CTF{Lp3_t0_Us3R_sUCc3SSfuLy}

