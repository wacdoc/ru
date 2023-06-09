Войдите в хранилище конфигурации ops.soft, запустите `./ssl.sh` , и в **верхнем каталоге** будет создана папка `conf` .

## преамбула

SMTP может напрямую приобретать услуги у поставщиков облачных услуг, таких как:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Облачная электронная почта Али](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Вы также можете создать свой собственный почтовый сервер - неограниченная отправка, низкая общая стоимость.

Ниже мы шаг за шагом демонстрируем, как создать собственный почтовый сервер.

## Выбор сервера

Для собственного SMTP-сервера требуется общедоступный IP-адрес с открытыми портами 25, 456 и 587.

Обычно используемые общедоступные облака заблокировали эти порты по умолчанию, и, возможно, их можно будет открыть, выпустив наряд на работу, но в конце концов это очень хлопотно.

Я рекомендую покупать у хоста, у которого открыты эти порты и который поддерживает настройку обратных доменных имен.

Здесь я рекомендую [Contabo](https://contabo.com) .

Contabo — это хостинг-провайдер, базирующийся в Мюнхене, Германия, основанный в 2003 году и предлагающий очень конкурентоспособные цены.

Если выбрать евро в качестве валюты покупки, цена будет дешевле (сервер с 8 ГБ памяти и 4 процессорами стоит около 529 юаней в год, а первоначальная плата за установку в течение одного года бесплатна).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

При размещении заказа отметьте `prefer AMD` , и сервер с процессором AMD будет иметь лучшую производительность.

Далее я возьму VPS Contabo в качестве примера, чтобы продемонстрировать, как создать собственный почтовый сервер.

## Конфигурация системы Ubuntu

Операционная система здесь Ubuntu 22.04.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Если сервер по ssh отображает `Welcome to TinyCore 13!` (как показано на рисунке ниже), это означает, что система еще не установлена. Отключите ssh и подождите несколько минут, чтобы снова войти в систему.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Когда появится сообщение `Welcome to Ubuntu 22.04.1 LTS` , инициализация завершена, и вы можете продолжить выполнение следующих шагов.

### [Необязательно] Инициализировать среду разработки

Этот шаг является необязательным.

Для удобства я разместил установку и системную настройку программного обеспечения ubuntu на [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Выполните следующую команду для установки одним щелчком мыши.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Пользователи из Китая, используйте вместо этого следующую команду, и язык, часовой пояс и т. д. будут установлены автоматически.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo включает IPV6

Включите IPV6, чтобы SMTP также мог отправлять электронные письма с адресами IPV6.

редактировать `/etc/sysctl.conf`

Измените или добавьте следующие строки

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Ознакомьтесь с [учебным пособием contabo: Добавление подключения IPv6 к вашему серверу](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Отредактируйте `/etc/netplan/01-netcfg.yaml` , добавьте несколько строк, как показано на рисунке ниже (файл конфигурации Contabo VPS по умолчанию уже содержит эти строки, просто раскомментируйте их).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Затем `netplan apply` , чтобы измененная конфигурация вступила в силу.

После успешной настройки вы можете использовать `curl 6.ipw.cn` для просмотра IPv6-адреса вашей внешней сети.

## Клонировать операции репозитория конфигурации

```
git clone https://github.com/wactax/ops.soft.git
```

## Создайте бесплатный SSL-сертификат для вашего доменного имени

Для отправки почты требуется SSL-сертификат для шифрования и подписи.

Мы используем [acme.sh](https://github.com/acmesh-official/acme.sh) для генерации сертификатов.

acme.sh — это инструмент автоматической подписи сертификатов с открытым исходным кодом,

Войдите в хранилище конфигурации ops.soft, запустите `./ssl.sh` , и в **верхнем каталоге** будет создана папка `conf` .

Найдите своего провайдера DNS из [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , отредактируйте `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Затем запустите `./ssl.sh 123.com` , чтобы сгенерировать сертификаты `123.com` и `*.123.com` для вашего доменного имени.

Первый запуск автоматически установит [acme.sh](https://github.com/acmesh-official/acme.sh) и добавит запланированную задачу для автоматического обновления. Вы можете увидеть `crontab -l` , там есть такая строка, как показано ниже.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Путь для сгенерированного сертификата выглядит примерно так `/mnt/www/.acme.sh/123.com_ecc。`

Обновление сертификата вызовет скрипт `conf/reload/123.com.sh` , отредактируйте этот скрипт, вы можете добавить такие команды, как `nginx -s reload` для обновления кеша сертификатов связанных приложений.

## Создайте SMTP-сервер с помощью chasquid

[chasquid](https://github.com/albertito/chasquid) — это SMTP-сервер с открытым исходным кодом, написанный на языке Go.

Как замена устаревшим программам почтового сервера, таким как Postfix и Sendmail, chasquid проще и удобнее в использовании, а также легче для вторичной разработки.

Запустите `./chasquid/init.sh 123.com` будет установлен автоматически одним щелчком мыши (замените 123.com на имя вашего отправляющего домена).

## Настройка подписи электронной почты DKIM

DKIM используется для отправки электронных подписей, чтобы письма не считались спамом.

После успешного выполнения команды вам будет предложено установить запись DKIM (как показано ниже).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Просто добавьте запись TXT в свой DNS (как показано ниже).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Просмотр состояния службы и журналов

 `systemctl status chasquid` Просмотр состояния службы.

Состояние нормальной работы показано на рисунке ниже.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` или `journalctl -xeu chasquid` может просматривать журнал ошибок.

## Обратная конфигурация доменного имени

Обратное доменное имя позволяет преобразовать IP-адрес в соответствующее доменное имя.

Установка обратного доменного имени может предотвратить идентификацию электронных писем как спама.

Когда почта получена, принимающий сервер выполнит обратный анализ доменного имени на IP-адресе отправляющего сервера, чтобы подтвердить, имеет ли отправляющий сервер действительное обратное доменное имя.

Если у отправляющего сервера нет обратного доменного имени или если обратное доменное имя не совпадает с IP-адресом отправляющего сервера, принимающий сервер может распознать электронное письмо как спам или отклонить его.

Посетите [https://my.contabo.com/rdns](https://my.contabo.com/rdns) и настройте, как показано ниже.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

После установки обратного доменного имени не забудьте настроить прямое разрешение доменного имени ipv4 и ipv6 на сервер.

## Отредактируйте имя хоста chasquid.conf

Измените `conf/chasquid/chasquid.conf` на значение обратного доменного имени.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Затем запустите `systemctl restart chasquid` , чтобы перезапустить службу.

## Бэкап конфига в репозиторий git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Например, я делаю резервную копию папки conf в свой собственный процесс github следующим образом.

Сначала создайте частный склад

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Войдите в каталог conf и отправьте на склад

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Добавить отправителя

бегать

```
chasquid-util user-add i@wac.tax
```

Можно добавить отправителя

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Убедитесь, что пароль установлен правильно

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

После добавления пользователя `chasquid/domains/wac.tax/users` обновится, не забудьте отправить его на склад.

## DNS добавить запись SPF

SPF (Sender Policy Framework) — это технология проверки электронной почты, используемая для предотвращения мошенничества с электронной почтой.

Он проверяет личность отправителя почты, проверяя, совпадает ли IP-адрес отправителя с записями DNS доменного имени, за которое он выдает себя, что предотвращает отправку мошенниками поддельных электронных писем.

Добавление записей SPF может максимально предотвратить идентификацию электронных писем как спама.

Если ваш сервер доменных имен не поддерживает тип SPF, просто добавьте запись типа TXT.

Например, SPF `wac.tax` выглядит следующим образом.

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF для `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Обратите внимание, что я включил здесь `include:_spf.google.com` , потому что позже я настрою `i@wac.tax` в качестве адреса отправки в почтовом ящике Google.

## Конфигурация DNS DMARC

DMARC — это аббревиатура от (Domain-based Message Authentication, Reporting & Conformance).

Он используется для захвата отказов SPF (могут быть вызваны ошибками конфигурации или кем-то другим, выдающим себя за вас, чтобы рассылать спам).

Добавьте запись TXT `_dmarc` ,

Содержание выглядит следующим образом

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Значение каждого параметра следующее

### р (Политика)

Указывает, как обрабатывать электронные письма, которые не проходят проверку SPF (структура политики отправителей) или DKIM (DomainKeys Identified Mail). Параметр p может быть установлен в одно из трех значений:

* нет: никаких действий не предпринимается, только результат проверки возвращается отправителю через механизм отчетов по электронной почте.
* Карантин: Поместите почту, не прошедшую проверку, в папку со спамом, но не отклоняйте почту напрямую.
* Отклонить: напрямую отклонять электронные письма, которые не прошли проверку.

### fo (параметры отказа)

Определяет количество информации, возвращаемой механизмом создания отчетов. Он может быть установлен в одно из следующих значений:

* 0: отчет о результатах проверки для всех сообщений.
* 1: сообщать только о сообщениях, не прошедших проверку
* d: сообщать только об ошибках проверки доменного имени
* s: сообщать только об ошибках проверки SPF
* l: сообщать только об ошибках проверки DKIM.

### руа и руф

* rua (Reporting URI для сводных отчетов): адрес электронной почты для получения сводных отчетов.
* ruf (Reporting URI для отчетов Forensic): адрес электронной почты для получения подробных отчетов.

## Добавьте записи MX для пересылки писем в Google Mail.

Поскольку я не смог найти бесплатный корпоративный почтовый ящик, поддерживающий универсальные адреса (Catch-All, может получать любые электронные письма, отправленные на это доменное имя, без ограничений на префиксы), я использовал chasquid для пересылки всех электронных писем в свой почтовый ящик Gmail.

**Если у вас есть собственный платный служебный почтовый ящик, не изменяйте MX и пропустите этот шаг.**

Отредактируйте `conf/chasquid/domains/wac.tax/aliases` , установите почтовый ящик для пересылки

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` указывает на все электронные письма, `i` — префикс адреса электронной почты отправляющего пользователя, созданный выше. Для пересылки почты каждому пользователю необходимо добавить строку.

Затем добавьте MX-запись (здесь я указываю непосредственно на адрес обратного доменного имени, как показано в первой строке на рисунке ниже).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

После завершения настройки вы можете использовать другие адреса электронной почты для отправки электронных писем на адреса `i@wac.tax` и `any123@wac.tax` , чтобы узнать, можете ли вы получать электронные письма в Gmail.

Если нет, проверьте журнал chasquid ( `grep chasquid /var/log/syslog` ).

## Отправьте электронное письмо на адрес i@wac.tax с помощью Google Mail.

После того, как Google Mail получил письмо, я, естественно, надеялся ответить с помощью `i@wac.tax` вместо i.wac.tax@gmail.com.

Посетите [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) и нажмите «Добавить другой адрес электронной почты».

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Затем введите проверочный код, полученный в письме, на которое было отправлено письмо.

Наконец, его можно установить в качестве адреса отправителя по умолчанию (вместе с возможностью ответа с тем же адресом).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Таким образом, мы завершили создание почтового сервера SMTP и в то же время используем Google Mail для отправки и получения электронных писем.

## Отправьте тестовое письмо, чтобы проверить, успешно ли настроена конфигурация.

Введите `ops/chasquid`

Запустите `direnv allow` установку зависимостей (direnv был установлен в предыдущем процессе инициализации с одним ключом, и в оболочку был добавлен хук)

затем беги

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Смысл параметров следующий

* пользователь: имя пользователя SMTP
* пароль: пароль SMTP
* кому: получатель

Вы можете отправить тестовое письмо.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Рекомендуется использовать Gmail для получения тестовых электронных писем, чтобы проверить, успешны ли настройки.

### Стандартное шифрование TLS

Как показано на рисунке ниже, этот небольшой замок означает, что SSL-сертификат был успешно включен.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Затем нажмите «Показать исходное письмо».

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### ДКИМ

Как показано на рисунке ниже, исходная почтовая страница Gmail отображает DKIM, что означает, что конфигурация DKIM выполнена успешно.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Проверьте Received в заголовке исходного письма, и вы увидите, что адрес отправителя — IPV6, что означает, что IPV6 также успешно настроен.
