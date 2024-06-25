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
