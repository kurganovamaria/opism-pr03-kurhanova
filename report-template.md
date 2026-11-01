<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ |Курганова Марія Григорівна |
| Група |F2 2.01 |
| Ідентифікатор (`sNN`) |se-201-17 |
| Дата виконання |01.11.2026 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu |

## Параметри варіанта

<!-- Переписати з картки stand/cards/sNN.md. -->

| Параметр | Значення |
|---|---|
| `$TTL`, с |30 |
| TTL запису SOA, с |45 |
| Поле `minimum` SOA, с |40 |
| Псевдонім (CNAME на `www`) | |
| Другий хост і його TTL, с | |
| Пріоритет MX | |
| Токен TXT | |
| Ім'я для завдання 5 | |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 |http | - |www.se-201-17.lab |80 |/users/archive |id=23&lang=asc |section-2 |
| 2 |http | - |files.se-201-17.lab |80 |/archive/ |id=%D0%BA%D0%B8%D1%97%D0%B2 | - |
| 3 |http | user:pass |stage.se-201-17.lab |80 |/users | - |comments |
| 4 |https | - |www.se-201-17.lab |4443 |/users/.../archive | - | - |
| 5 |mailto | - | - | - |admin@se-201-17.lab |subject=users | - |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 |www.se-201-17.lab |www.se-201-17.lab |users/archive?id=23&lang=asc#section-2 |
| 2 |files.se-201-17.lab |files.se-201-17.lab |archive/?id=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 |stage.se-201-17.lab |stage.se-201-17.lab |/users |
| 4 |www.se-201-17.lab |www.se-201-17.lab:4443 |/users/.../archive |
| 5 |se-201-17.lab | - | - |

Декодоване значення параметра URI 2:

Чому URI 5 не має складової host, хоча містить символ `@`:

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->

```text
root@ns-se-201-17:/# curl -v "http://www.se-201-17.lab/users/archive?id=23&lang=asc#section-2"
* Host www.se-201-17.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.se-201-17.lab (10.53.0.80) port 80
> GET /users/archive?id=23&lang=asc HTTP/1.1
> Host: www.se-201-17.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 13:13:57 GMT
< Content-Type: text/plain
< Content-Length: 142
< Connection: keep-alive
< 
server: web-a
Host: www.se-201-17.lab
request URI: /users/archive?id=23&lang=asc
path: /users/archive
query: id=23&lang=asc
client: 10.53.1.1
* Connection #0 to host www.se-201-17.lab left intact
root@ns-se-201-17:/# curl -v "http://files.se-201-17.lab:80/archive/?id=%D0%BA%D0%B8%D1%97%D0%B2"
* Host files.se-201-17.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to files.se-201-17.lab (10.53.0.80) port 80
> GET /archive/?id=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: files.se-201-17.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 13:14:04 GMT
< Content-Type: text/plain
< Content-Length: 160
< Connection: keep-alive
< 
server: web-a
Host: files.se-201-17.lab
request URI: /archive/?id=%D0%BA%D0%B8%D1%97%D0%B2
path: /archive/
query: id=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host files.se-201-17.lab left intact
root@ns-se-201-17:/# curl -v "http://user:pass@stage.se-201-17.lab/users#comments"
* Host stage.se-201-17.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to stage.se-201-17.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /users HTTP/1.1
> Host: stage.se-201-17.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Sun, 01 Nov 2026 13:14:15 GMT
< Content-Type: text/plain
< Content-Length: 99
< Connection: keep-alive
< 
server: web-b
Host: stage.se-201-17.lab
request URI: /users
path: /users
query: 
client: 10.53.1.1
* Connection #0 to host stage.se-201-17.lab left intact
```

Значення заголовка `Authorization` і результат декодування:

```text
> Authorization: Basic dXNlcjpwYXNz
* Connection #0 to host stage.se-201-17.lab left intact
root@ns-se-201-17:/# echo 'dXNlcjpwYXNz' | base64 -d
user:passroot@ns-se-201-17:/# 
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

```text
root@ns-se-201-17:/# curl -v https://www.se-201-17.lab:4443/users/../archive./archive
* Host www.se-201-17.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:4443...
* connect to 10.53.0.80 port 4443 from 10.53.1.1 port 37536 failed: Connection refused
* Failed to connect to www.se-201-17.lab port 4443 after 1 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-17.lab port 4443 after 1 ms: Couldn't connect to server
root@ns-se-201-17:/# curl -v https://www.se-201-17.lab:4443/users/../archive
* Host www.se-201-17.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:4443...
* connect to 10.53.0.80 port 4443 from 10.53.1.1 port 57822 failed: Connection refused
* Failed to connect to www.se-201-17.lab port 4443 after 0 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-17.lab port 4443 after 0 ms: Couldn't connect to server
root@ns-se-201-17:/# 
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний |/archive |/archive |
| `--path-as-is` |/users/../archive |/archive |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:
Під час виконання звичайного запиту утиліта curl за замовчуванням самостійно виконує нормалізацію URL-шляху перед відправкою на сервер, видаляючи конструкції переходу на рівень вище на кшталт /users/../, через що сервер отримує вже скорочений шлях і request URI, які дорівнюють /archive. Водночас запит із прапорцем --path-as-is забороняє клієнту виконувати нормалізацію шляху, внаслідок чого запит передається на сервер в оригінальному вигляді зі збереженням усієї послідовності символів (/users/../archive), і сервер фіксує цей шлях у незмінному вигляді як у request URI, так і в path.

## Завдання 2. Проходження ієрархії вручну

```text
root@ns-se-201-17:/# dig @10.53.0.2 ns.se-201-17.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.se-201-17.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26353
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8ecd707d75cd0e03010000006ae725c642fb406f615e7010 (good)
;; QUESTION SECTION:
;ns.se-201-17.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 79 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Sun Nov 01 11:43:02 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-se-201-17:/# dig @10.53.0.3 ns.se-201-17.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.se-201-17.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59436
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e4db22a8c9b5ef28010000006ae725ce674dc7f9391a6da7 (good)
;; QUESTION SECTION:
;ns.se-201-17.lab.              IN      A

;; AUTHORITY SECTION:
se-201-17.lab.          3600    IN      NS      ns.se-201-17.lab.

;; ADDITIONAL SECTION:
ns.se-201-17.lab.       3600    IN      A       10.53.1.1

;; Query time: 40 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sun Nov 01 11:43:10 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-se-201-17:/# dig @10.53.1.1 ns.se-201-17.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.se-201-17.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48410
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6a13493038f93e1f010000006ae725dbf357d5fd81313762 (good)
;; QUESTION SECTION:
;ns.se-201-17.lab.              IN      A

;; ANSWER SECTION:
ns.se-201-17.lab.       30      IN      A       10.53.1.1

;; AUTHORITY SECTION:
se-201-17.lab.          30      IN      NS      ns.se-201-17.lab.

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 11:43:23 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 |NOERROR |qr |0 |1 |2 |
| 10.53.0.3 |NOERROR |qr |0 |1 |2 |
| 10.53.1.1 |NOERROR |qr aa |1 |1 |1 |

Вивід `dig ns.sNN.lab A +trace`:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.se-201-17.lab A +trace
;; global options: +cmd
.                       83720   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-201-17.lab.          3600    IN      NS      ns.se-201-17.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 39 ms

ns.se-201-17.lab.       30      IN      A       10.53.1.1
se-201-17.lab.          30      IN      NS      ns.se-201-17.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.se-201-17.lab) in 0 ms

```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` |3600 |Немає |AUTHORITY |
| `dig @10.53.1.1 sNN.lab NS +norec` |30 |Є |ANSWER |

1. Які відповіді є відсиланнями і за якими ознаками це визначено?
Відповіді від серверів 10.53.0.2 та 10.53.0.3 є відсиланнями, оскільки вони не містять записів ANSWER для запитуваного імені, а лише вказують на інші сервери (NS) у секції AUTHORITY. 
2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
У секції ADDITIONAL присутній запис A для NS-сервера, що дозволяє резолверу одразу отримати IP-адресу для подальшого запиту без необхідності робити додатковий запит на отримання цієї адреси. Якби цього запису не було, резолверу довелося б робити ще один запит для отримання IP-адреси NS-сервера, що збільшило б час відповіді.
3. Яка з двох копій запису NS є авторитативною і чому?
Авторитативною є копія на сервері 10.53.1.1, оскільки вона міститься у секції ANSWER і має прапорець aa.
## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```text
root@ns-se-201-17:/# zone-reload
zone se-201-17.lab/IN: loaded serial 2026110102
OK
zone reload queued
```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A |NOERROR |Є |10.53.0.80 |30 |так |
| псевдонім CNAME |NOERROR |Є |www.se-201-17.lab. |30 |так |
| `mail` A |NOERROR |Є |10.53.0.80 |30 |так |
| `@` MX |NOERROR |Є |35 mail.se-201-17.lab. |30 |так |
| `@` TXT |NOERROR |Є |"lab-token=b22d1046" |30 |так |
| другий хост A |NOERROR |Є |10.53.0.81 |40 |так |

Порівняння відповідей резолвера та власного сервера:

```text
root@ns-se-201-17:/# dig files.se-201-17.lab

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> files.se-201-17.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16977
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;files.se-201-17.lab.           IN      A

;; ANSWER SECTION:
files.se-201-17.lab.    30      IN      CNAME   www.se-201-17.lab.
www.se-201-17.lab.      30      IN      A       10.53.0.80

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 12:43:42 UTC 2026
;; MSG SIZE  rcvd: 82

root@ns-se-201-17:/# dig @10.53.1.1 files.se-201-17.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 files.se-201-17.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12744
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: baca33370595102a010000006ae7340646e89fcc60f3df58 (good)
;; QUESTION SECTION:
;files.se-201-17.lab.           IN      A

;; ANSWER SECTION:
files.se-201-17.lab.    30      IN      CNAME   www.se-201-17.lab.
www.se-201-17.lab.      30      IN      A       10.53.0.80

;; AUTHORITY SECTION:
se-201-17.lab.          30      IN      NS      ns.se-201-17.lab.

;; ADDITIONAL SECTION:
ns.se-201-17.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:43:50 UTC 2026
;; MSG SIZE  rcvd: 143

root@ns-se-201-17:/# dig se-201-17.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> se-201-17.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29078
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;se-201-17.lab.                 IN      MX

;; ANSWER SECTION:
se-201-17.lab.          30      IN      MX      35 mail.se-201-17.lab.

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Sun Nov 01 12:45:14 UTC 2026
;; MSG SIZE  rcvd: 63

root@ns-se-201-17:/# dig @10.53.1.1 se-201-17.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 se-201-17.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19794
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c49b5ca3f176b993010000006ae734696608439ed236ff7c (good)
;; QUESTION SECTION:
;se-201-17.lab.                 IN      MX

;; ANSWER SECTION:
se-201-17.lab.          30      IN      MX      35 mail.se-201-17.lab.

;; AUTHORITY SECTION:
se-201-17.lab.          30      IN      NS      ns.se-201-17.lab.

;; ADDITIONAL SECTION:
mail.se-201-17.lab.     30      IN      A       10.53.0.80
ns.se-201-17.lab.       30      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 12:45:29 UTC 2026
;; MSG SIZE  rcvd: 140

```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a |qr rd ra |2 |0 |1 |30 |
| псевдонім через 10.53.1.1 |qr aa |2 |1 |2 |30 |
| MX через resolver-a |qr rd ra |1 |0 |1 |30 |
| MX через 10.53.1.1 |qr aa |1 |1 |3 |30 |

Пояснення відмінностей:

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```text
root@ns-se-201-17:/# zone-reload
dns_master_load: /lab/zone/se-201-17.lab.zone:15: se-201-17.lab: CNAME and other data
dns_master_load: /lab/zone/se-201-17.lab.zone:15: se-201-17.lab: CNAME and other data
dns_master_load: /lab/zone/se-201-17.lab.zone:20: se-201-17.lab: CNAME and other data
dns_master_load: /lab/zone/se-201-17.lab.zone:20: se-201-17.lab: CNAME and other data
zone se-201-17.lab/IN: loading from master file /lab/zone/se-201-17.lab.zone failed: CNAME and other data
zone se-201-17.lab/IN: not loaded due to errors.
```

1. Який TTL має кожен запис зони і звідки це значення взято?
Запис SOA має чітко прописаний TTL у 45 секунд. Для всіх інших стандартних записів (www, files, mail, MX, TXT і базових NS) час життя становить 30 секунд - вони просто підтягують його за замовчуванням з глобальної директиви $TTL 30, яка задана на початку файлу. А от для запису stage заданий індивідуальний TTL прямо в його рядку, тому там стоїть 40 секунд.


2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?
У секції ANSWER падає два записи: спочатку сам CNAME для files (який вказує на www), а одразу за ним - A-запис www з нашою адресою 10.53.0.80.

3. Чому запис CNAME неможливо розмістити для імені `@`?
Тому що @ - це ім'я зони se-201-17.lab, і згідно з правилами DNS, CNAME не може співіснувати з іншими записами для того ж імені. У цьому випадку для @ вже існують записи MX і TXT, тому додавання CNAME призведе до конфлікту.

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text

```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text

```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text

```

Очікуваний час появи нової адреси (підставити R у формулу):

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text

```

Рядок `server:` з відповіді `curl`:

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| | | | | |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text

```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text

```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text

```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| | | | | |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | | |
| B: нове ім'я в зоні `lab.` | | |
| C: ім'я в неіснуючому TLD | | |
| D: `www.sNN.lab` після перезапуску резолвера | | |

Вивід вимірювань:

```text

```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | | | |
| B | | | |
| C | | | |
| D | | | |

Значення `time_namelookup` з `curl` після перезапуску резолвера:

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text

```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text

```

Запити із заміром часу (крок 5):

```text

```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | | | |
| resolver-a, другий | | | |
| resolver-b | | | |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text

```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text

```

### Постмортем

1. Що сталося:
2. Вплив на клієнтів resolver-a і resolver-b:
3. Хронологія з виміряним часом:
4. Чому кеш допоміг або не допоміг:
5. Що слід змінити:

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text

```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:

## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні -->

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | | | |
