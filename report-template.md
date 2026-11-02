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
| Псевдонім (CNAME на `www`) |files |
| Другий хост і його TTL, с |stage, 40 |
| Пріоритет MX |10 |
| Токен TXT |se-201-17-token |
| Ім'я для завдання 5 |mail |

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
root@ns-se-201-17:/# dig www.se-201-17.lab +noall +answer
www.se-201-17.lab.      30      IN      A       10.53.0.81
root@ns-se-201-17:/# dig www.se-201-17.lab +noall +answer
www.se-201-17.lab.      18      IN      A       10.53.0.81
```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка: Формула часу появи нової адреси через залишок R Tочікуваний = Tзміни + R, Найбільша можлива затримка: 30 c

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-se-201-17:/# zone-edit
root@ns-se-201-17:/# dig @10.53.1.1 www.se-201-17.lab +norec +short
10.53.0.80
```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```text
root@ns-se-201-17:/# date +%T; dig www.se-201-17.lab +noall +answer
15:17:22
www.se-201-17.lab.      30      IN      A       10.53.0.80
root@ns-se-201-17:/# zone-reload
zone se-201-17.lab/IN: loaded serial 2026110111
OK
zone reload queued
root@ns-se-201-17:/# date +%T
15:18:47
root@ns-se-201-17:/# dig @10.53.1.1 www.se-201-17.lab +norec +short
10.53.0.81
```

Очікуваний час появи нової адреси (підставити R у формулу): 15:18:47 + 30c = 15:19:17

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
root@ns-se-201-17:/# while true; do echo "$(date +%T) $(dig +short www.se-201-17.lab)"; sleep 2; done
15:19:05 10.53.0.80
15:19:07 10.53.0.80

15:19:11 10.53.0.80
15:19:13 10.53.0.80
15:19:15 10.53.0.81
^C
```

Рядок `server:` з відповіді `curl`:

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
|15:19:05 |15:19:17 |15:19:15 |2 |30 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

​Розбіжність між прогнозом (30 секунд) і фактичним виміряним значенням затримки пояснюється тим, що на момент виконання команди zone-reload запис уже певний час перебував у кеші резолвера. Відповідно, залишок терміну життя запису (R) на той момент був меншим за повний початковий TTL.

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

Найбільша можлива затримка становила б 30 секунд відповідно до базового значенню TTL за замовчуванням. Вона виникає за умови, що зміна на авторитативному сервері (zone-reload) виконується одразу після того, як резолвер щойно закешував запис із повним початковим терміном життя.

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

Завчасно (за 24 години) зменшити значення TTL цього запису у файлі зони до невеликого (наприклад, 300 секунд). Дочекатися закінчення дії старого великого TTL на резолверах, змінити IP-адресу та виконати zone-reload. Після успішного перенесення стандартний TTL можна повернути назад.

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення: min(SOA TTL, SOA minimum) = min(45,40) = 40 c

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
root@ns-se-201-17:/# dig @10.53.1.1 new-jzyqu.se-201-17.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-jzyqu.se-201-17.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 35403
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 52790996628ba91c010000006ae82b2c6eeabeb80dd6c0f1 (good)
;; QUESTION SECTION:
;new-jzyqu.se-201-17.lab.       IN      A

;; AUTHORITY SECTION:
se-201-17.lab.          40      IN      SOA     ns.se-201-17.lab. admin.se-201-17.lab. 2026110101 3600 600 86400 40

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 06:18:20 UTC 2026
;; MSG SIZE  rcvd: 125

```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text
root@ns-se-201-17:/# zone-edit
root@ns-se-201-17:/# dig @10.53.1.1 new3-jzyqu.se-201-17.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new3-jzyqu.se-201-17.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 35965
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 8957296b3c9097b3010000006ae82fb47202f67e3b908812 (good)
;; QUESTION SECTION:
;new3-jzyqu.se-201-17.lab.      IN      A

;; AUTHORITY SECTION:
se-201-17.lab.          40      IN      SOA     ns.se-201-17.lab. admin.se-201-17.lab. 2026110103 3600 600 86400 40

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 06:37:40 UTC 2026
;; MSG SIZE  rcvd: 126

root@ns-se-201-17:/# date +%T; dig @10.53.0.53 new3-jzyqu.se-201-17.lab
06:37:53

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 new3-jzyqu.se-201-17.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 64086
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new3-jzyqu.se-201-17.lab.      IN      A

;; AUTHORITY SECTION:
se-201-17.lab.          40      IN      SOA     ns.se-201-17.lab. admin.se-201-17.lab. 2026110103 3600 600 86400 40

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 06:37:53 UTC 2026
;; MSG SIZE  rcvd: 98

root@ns-se-201-17:/# zone-reload; date +%T
zone se-201-17.lab/IN: loaded serial 2026110104
OK
zone reload queued
06:38:02
root@ns-se-201-17:/# dig @10.53.1.1 new3-jzyqu.se-201-17.lab +norec +short
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
root@ns-se-201-17:/# while true; do echo "$(date +%T) $(dig @10.53.0.53 new3-jzyqu.se-201-17.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
06:38:26 status: NXDOMAIN
06:38:28 status: NXDOMAIN
06:38:32 status: NXDOMAIN
06:38:34 status: NOERROR
^C
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
|06:37:53 |06:38:02 |06:38:34 |32 |40 |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

Під час виконання використовувалося ім'я new3-jzyqu.se-201-17.lab. Причиною повторення з іншим ім'ям стало те, що під час попередніх спроб негативний кеш спливав до завершення повноцінного циклу моніторингу, через що резолвер передчасно починав повертати статус NOERROR.

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

​Оскільки запитуваний неіснуючий ресурс фізично відсутній у зоні, для нього неможливо задати окремий власний TTL.

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

​Ні, не збігається. У файлі зони вказано статичні базові параметри значення $TTL та поле minimum, на основі яких розраховується час негативного кешування. Натомість у секції AUTHORITY відповіді резолвера цей TTL є динамічним він відображає залишок часу життя запису в кеші і поступово зменшується в реальному часі.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

Одразу після створення запису скрипт побачить статус NXDOMAIN, оскільки кешуючий резолвер продовжує віддавати старий негативний кеш. Щоб це виправити, слід звертатися напряму до авторитативного сервера або закласти у сценарій примусову паузу sleep, яка перекриває час дії негативного TTL.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні |10.53.1.1 |Кеш резолвера |
| B: нове ім'я в зоні `lab.` |10.53.0.3 |Кеш резолвера |
| C: ім'я в неіснуючому TLD |10.53.0.2 |Кеш резолвера |
| D: `www.sNN.lab` після перезапуску резолвера |10.53.0.2/10.53.0.3/10.53.1.1 |Кеш резолвера |

Вивід вимірювань:

```text
root@ns-se-201-17:/# A=y$RANDOM.se-201-17.lab; dig @10.53.0.53 $A | grep 'Query time'; dig @10.53.0.53 $A | grep 'Query time'
;; Query time: 0 msec
;; Query time: 0 msec
root@ns-se-201-17:/# B=x$RANDOM.lab; dig @10.53.0.53 $B | grep 'Query time'; dig @10.53.0.53 $B | grep 'Query time'
;; Query time: 41 msec
;; Query time: 0 msec
root@ns-se-201-17:/# C=x$RANDOM.zz$RANDOM; dig @10.53.0.53 $C | grep 'Query time'; dig @10.53.0.53 $C | grep 'Query time'
;; Query time: 81 msec
;; Query time: 0 msec
root@ns-se-201-17:/# dig www.se-201-17.lab | grep 'Query time'
;; Query time: 414 msec
root@ns-se-201-17:/# dig www.se-201-17.lab | grep 'Query time'
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A |0 |0 |10.53.1.1 |
| B |41 |0 |10.53.0.3 |
| C |81 |0 |10.53.0.2 |
| D |414 |0 |10.53.0.2/10.53.0.3/10.53.1.1 |

Значення `time_namelookup` з `curl` після перезапуску резолвера: 0.402920

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text
root
02-Nov-2026 16:13:59.486 client ... 10.53.0.53#62162 (.): query: . IN NS -E(0)DC (10.53.0.2)
02-Nov-2026 16:13:59.566 client ... 10.53.0.53#29141 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
02-Nov-2026 16:13:59.646 client ... 10.53.0.53#50341 (www.se-201-17.lab): query: www.se-201-17.lab IN A -E(0)DC (10.53.0.2)

TLD
02-Nov-2026 16:13:59.768 client ... 10.53.0.53#31459 (www.se-201-17.lab): query: www.se-201-17.lab IN A -E(0)DC (10.53.0.3)

Власний сервер
02-Nov-2026 16:13:59.808 client ... 10.53.0.53#17809 (www.se-201-17.lab): query: www.se-201-17.lab IN A -E(0)DC (10.53.1.1)
```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

Root-сервер вносить затримку близько 81 мс, що безпосередньо витікає з рядка C для запиту в неіснуючий TLD. TLD-сервер додає приблизно 41 мс, що видно з рядка B для зони .lab. Власний авторитативний сервер забезпечує мінімальну затримку, що випливає з рядка А завдяки використанню раніше закешованого делегування.

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

У рядку А перший запит не звертається до root і TLD, оскільки резолвер уже зберігає у своєму кеші NS-записи для зони se-201-17.lab. Через це він одразу знає IP-адресу авторитативного сервера та спрямовує запит безпосередньо до нього, минаючи root і TLD.

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

Завантаження ресурсів із п'яти різних доменів суттєво збільшує час першого відкриття сторінки. Для кожного нового домену резолвер змушений незалежно виконувати пошук через root, TLD та авторитативні сервери. У результаті затримки DNS-запитів сумуються, що помітно сповільнює загальну ініціалізацію та відображення вебсторінки.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь: 

Після зупинки resolver-a та спливання TTL cпроба оновити його призведе до звернення на недоступний сервер тому резолвер поверне помилку (SERVFAIL) або станеться таймаут. В той час як resolver-b навіть після спливання TTL і зупинки авторитативного сервера повинен повернути застарілий запис із кешу за декілька мс.


Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text
root@ns-se-201-17:/# dig @10.53.0.53 stage.se-201-17.lab +noall +answer; dig @10.53.0.54 stage.se-201-17.lab +noall +answer
stage.se-201-17.lab.    40      IN      A       10.53.0.81
stage.se-201-17.lab.    40      IN      A       10.53.0.81
root@ns-se-201-17:/# named-stop; date +%T
16:52:22
```

Запити із заміром часу (крок 5):

```text
root@ns-se-201-17:/# time dig @10.53.0.53 stage.se-201-17.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 stage.se-201-17.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.041s
user    0m0.006s
sys     0m0.005s
root@ns-se-201-17:/# time dig @10.53.0.53 stage.se-201-17.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 stage.se-201-17.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.052s
user    0m0.006s
sys     0m0.006s
root@ns-se-201-17:/# time dig @10.53.0.54 stage.se-201-17.lab +tries=1 +timeout=30

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 stage.se-201-17.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55074
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;stage.se-201-17.lab.           IN      A

;; ANSWER SECTION:
stage.se-201-17.lab.    30      IN      A       10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Mon Nov 02 16:58:59 UTC 2026
;; MSG SIZE  rcvd: 64


real    0m1.826s
user    0m0.006s
sys     0m0.007s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший |timed out | - |0m30.041s |
| resolver-a, другий |timed out | - |0m30.052s |
| resolver-b |NOERROR |stage.se-201-17.lab.    30      IN      A       10.53.0.81 |0m1.826s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
root@ns-se-201-17:/# named-start
root@ns-se-201-17:/# date +%T
17:03:06
root@ns-se-201-17:/# while true; do echo "$(date +%T) $(dig @10.53.0.53 stage.se-201-17.lab +short +tries=1 +timeout=3)"; sleep 2; done
17:03:56 
17:03:58 
17:04:00 
17:04:02 
17:04:04 
17:04:06 10.53.0.81
^C
```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text
root@ns-se-201-17:/# dig @10.53.1.1 se-201-17.lab SOA +norec +short
ns.se-201-17.lab. admin.se-201-17.lab. 2026110111 3600 600 86400 40
```

### Постмортем

1. Що сталося: Авторитативний сервер ns-se-201-17 було примусово зупинено, а TTL для запису вичерпався.
2. Вплив на клієнтів resolver-a і resolver-b: Клієнти resolver-a зіткнулися з повним припиненням обслуговування (таймаути по 30 секунд), тоді як клієнти resolver-b продовжували отримувати дані без перебоїв
3. Хронологія з виміряним часом: О 16:52:22 зупинено сервер; після спливання 30 с TTL запити до resolver-a завершилися тайм-аутом, а resolver-b відповів за 1.8 с, після named-start робота повністю відновилася.
4. Чому кеш допоміг або не допоміг: Стандартний кеш resolver-a не допоміг, оскільки після закінчення TTL запис був видалений, а оновлення з недоступного сервера неможливе. Кеш у resolver-b допоміг завдяки використанню застарілих даних.
5. Що слід змінити: Для критично важливих зон варто впроваджувати резервні авторитативні сервери та налаштовувати рекурсивні резолвери з підтримкою режиму serve-stale..

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

У відповіді resolver-b запис мав застарілий TTL (30 секунд). Такий TTL вказує клієнтам на те, що інформація не є свіжою, проте дозволяє уникнути збоїв у додатках під час аварій авторитативного джерела.

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

Обидва запити до resolver-a завершилися однаковим тайм-аутом тривалістю близько 30 секунд. Протягом цього часу резолвер намагався встановити зв'язок із недоступним авторитативним сервером згідно з параметром таймауту, чекаючи на відповідь перед поверненням помилки.


3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

Ім'я через resolver-a зарезолвилося не миттєво через те, що його кеш був повністю очищений після попереднього таймауту і збою. Резолверу знадобився короткий час на виконання нового ієрархічного запиту до щойно запущеного авторитативного сервера.


4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

Режим serve-stale шкодить у ситуаціях, коли на авторитативному сервері відбулася планова зміна критичних даних. У такому разі резолвер тривалий час продовжуватиме роздавати стару, неактуальну адресу з кешу замість нової, що призведе до помилок підключення клієнтів.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text
root@ns-se-201-17:/# cat /lab/zone/se-201-17.lab.zone
; Zone se-201-17.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 30
@     45 IN SOA ns.se-201-17.lab. admin.se-201-17.lab. (
            2026110111 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            40         ; minimum
        )
@     IN NS ns.se-201-17.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www     IN      A       10.53.0.81
files   IN      CNAME   www
mail    IN      A       10.53.0.80
@       IN      MX      35 mail
@       IN      TXT     "lab-token=b22d1046"
stage   40      IN      A       10.53.0.81
new-jzyqu    IN    A    10.53.0.80
new2-jzyqu   IN   A   10.53.0.80
new3-jzyqu   IN   A   10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно: 

Під час випробування режиму serve-stale у завданні 7 стало несподіванкою те, що запит до resolver-b тривав близько 1.8 секунди (real 0m1.826s), хоча очікувалося мілісекундне реагування кешу.


**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося: Не вдалося остаточно пояснити, чому час відповіді кореневого сервера ;; Query time: 81 msec` у рядку C так суттєво перевищує затримку локального авторитативного сервера. Попри аналіз специфікацій ієрархічного резолвінгу, з'ясувати, чи зумовлена ця різниця середовищем Docker, чи внутрішньою логікою BIND 9, на практиці не вдалося.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 |Gemini 3 |Сформулюй висновки для лабораторної роботи з DNS щодо затримки кореневих серверів |Використано загальну структуру та логіку пояснення, самостійно виконано практичні вимірювання на стенді, підставлено власні дані та рядки виводу. |
| 2 |Gemini 3 |Допоможи сформулювати постмортем для завдання 7 з зупинкою власного сервера. |Використано логіку опису інциденту, час, назву хоста та деталі впливу на клієнтів адаптовано під власні результати виконання на стенді. |
| 3 |Gemini 3 | Як пояснити різницю між таймаутом resolver-a та відповіддю resolver-b у завданні 7?|Взято за основу теоретичне пояснення механізму serve-stale, самостійно перевірено та зафіксовано фактичні значення real і Query time з виводу термінала.|
