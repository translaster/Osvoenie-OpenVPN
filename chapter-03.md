# Глава 3. PKI и сертификаты

В первую очередь OpenVPN использует сертификаты X.509 для аутентификации клиента и шифрования трафика VPN, хотя эту поддержку можно отключить. Просмотр списка рассылки и истории IRC-каналов, настройка и обслуживание **инфраструктуры приватного ключа (PKI)** для сертификатов X.509 является сложной концепцией и может быть трудоемкой задачей.

Двоичный файл OpenSSL содержит все инструменты, необходимые для ручного управления PKI, но параметры команды сложны и если не автоматизированы, могут быть подвержены ошибкам. Организациям или частным лицам рекомендуется использовать сценарий или другой пакет для управления своей PKI. Это не только ограничивает количество ошибок, но также позволяет лучше соблюдать правила и другие общие критерии.

Существуют два проекта с открытым исходным кодом, которые специально написаны для хорошей работы с реализациями OpenVPN. EasyRSA - это давний проект, который всегда был тесно связан с проектом OpenVPN. Первоначально написанный вместе с OpenVPN, его первоначальная цель заключалась в создании **центра сертификации (Certificate Authority - CA)** и необходимых компонентов. Сегодня этот проект все еще поддерживается вместе с проектом OpenVPN, хотя они технически разделены.

Другой проект - ssl-admin, представляет собой Perl-скрипт, написанный для заполнения видимых пробелов в коде EasyRSA. Оба проекта по-разному подходят к задачам управления PKI и оба имеют уникальное решение. Проект ssl-admin - это интерактивный скрипт, предоставляющий меню и отзывы пользователей, в то время как EasyRSA - это, прежде всего, пакетная утилита.

На сегодняшний день Эрик Крист поддерживает проекты EasyRSA и ssl-admin. Джош Сепек присоединился и написал большую часть исходного кода EasyRSA v3.0. Цель состоит в том, чтобы в конечном итоге объединить два проекта и сохранить все функциональные возможности обоих.

## Обзор PKI

PKI - это, как правило, иерархическая организация сертификатов шифрования и пар ключей. Как правило, как и в большинстве веб-сайтов, вершиной иерархии является CA. Это _корень_ всего дерева, и доверие основано на этом уровне. Если корень является доверенным, все пары ключей, лежащие в основе, также будут доверенными. Из CA корневого уровня могут быть клиентские сертификаты, серверные сертификаты, подчиненные CA и **списки отзыва сертификатов (certificate revocation lists - CRL)**. Под каждым подчиненным CA этот список возможностей повторяется.

![](pics/pic3-1.png)

Чтобы использовать PKI в полной мере, пользователи и системы должны доверять корневому ЦС и любым промежуточным ЦС в цепочке. В большинстве современных веб-браузеров авторы или поставщики браузера по умолчанию проверяют и утверждают большой список центров сертификации корневого уровня, которым можно доверять. Этими органами обычно являются коммерческие поставщики, такие как VeriSign, Go Daddy, Comodo, Trend Micro, различные государственные структуры и многие другие.

Из-за этого предварительно одобренного списка для браузеров, подавляющее большинство пользователей Интернета совершенно не знают о том, как работает PKI. В результате, новые пользователи OpenVPN и любое программное обеспечение, которому требуется PKI, находят настройку и управление препятствием - как концептуально, так и технически. В случае общедоступных веб-сайтов требуется проверка стороннего сайта доверенным органом. Однако в контексте OpenVPN, как правило, в организации существует единый объект, которому неявно доверяют, отдел IT. OpenVPN обычно используется в одной организации, и доверие к нему является неотъемлемым. EasyRSA и ssl-admin были написаны, чтобы помочь как начинающим, так и опытным пользователям лучше управлять своей PKI.

При использовании единственной двухточечной связи часто не имеет смысла включать сложность PKI для защиты туннеля; предустановленных общих ключей достаточно. Однако, когда в дело вовлечено много пользователей, существует гораздо больший потенциал для потерянных и украденных ключей и текучести кадров. При правильно настроенной PKI отозвать утерянный сертификат или увольняющийся сотрудник относительно просто. Новый также легко создается и перераспределяется.

Используя EasyRSA и ssl-admin, мы создадим простую PKI с CA, сертификатом сервера, некоторыми сертификатами клиента и списком отзыва сертификатов. Кроме того, мы будем использовать эти утилиты для генерации параметров **Диффи-Хеллмана (DH)**, которые будут использоваться и обсуждаться в последующих главах.

Поток OpenVPN PKI выглядит следующим образом:

![](pics/pic3-2.png)

### PKI с использованием EasyRSA

На момент написания этой главы EasyRSA 2.2.2 была официально последней версией наряду с v3.0.0-rc2. Поскольку серия v3.0 близится к выпуску, этот раздел будет посвящен этой версии. Релизы для EasyRSA можно найти по адресу https://github.com/OpenVPN/easyrsa/releases.

Это упражнение продемонстрирует как построить ЦС с нуля. Обновление с EasyRSA 2.2 здесь не рассматривается. После загрузки пакета EasyRSA распакуйте файлы, и вы должны найти каталог (в нашем случае, `EasyRSA-3.0.0-rc2`):

```
ecrist@computer:~/Downloads-> tar -xzvf EasyRSA-3.0.0-rc2.tgz
x EasyRSA-3.0.0-rc2/
x EasyRSA-3.0.0-rc2/x509-types/
x EasyRSA-3.0.0-rc2/x509-types/server
x EasyRSA-3.0.0-rc2/x509-types/ca
x EasyRSA-3.0.0-rc2/x509-types/COMMON
x EasyRSA-3.0.0-rc2/x509-types/client
x EasyRSA-3.0.0-rc2/openssl-1.0.cnf
x EasyRSA-3.0.0-rc2/ChangeLog
x EasyRSA-3.0.0-rc2/Licensing/
x EasyRSA-3.0.0-rc2/Licensing/gpl-2.0.txt
x EasyRSA-3.0.0-rc2/COPYING
x EasyRSA-3.0.0-rc2/KNOWN_ISSUES
x EasyRSA-3.0.0-rc2/doc/
x EasyRSA-3.0.0-rc2/doc/Hacking.md
x EasyRSA-3.0.0-rc2/doc/EasyRSA-Upgrade-Notes.md
x EasyRSA-3.0.0-rc2/doc/EasyRSA-Readme.md
x EasyRSA-3.0.0-rc2/doc/EasyRSA-Advanced.md
x EasyRSA-3.0.0-rc2/doc/Intro-To-PKI.md
x EasyRSA-3.0.0-rc2/README.quickstart.md
x EasyRSA-3.0.0-rc2/vars.example
x EasyRSA-3.0.0-rc2/easyrsa
```

После извлечения, скопируйте файл `vars.example` в `vars`. Рекомендуется установить рабочий каталог EasyRSA. Строка 45 из `vars` определяет `EASYRSA` в `$PWD` (текущий рабочий каталог) по умолчанию. Это может быть проблематично, особенно если вы используете EasyRSA для управления несколькими центрами сертификации. Раскомментируйте эту строку и измените ее на что-то подходящее для вашей среды, например, `/usr/local/etc/easyrsa`.

---

**Подсказка**
При инициализации EasyRSA все содержимое каталога `EASYRSA` будет удалено. Будьте осторожны с тем, что вы определяете здесь. В каталоге `EASYRSA` находится хранилище сертификатов. Это _не_ где исполняемые файлы и переменные.

---

Вы, вероятно, захотите установить организационные поля, которые являются следующими переменными:

* `EASYRSA_REQ_COUNTRY`
* `EASYRSA_REQ_PROVICE`
* `EASYRSA_REQ_CITY`
* `EASYRSA_REQ_ORG`
* `EASYRSA_REQ_EMAIL`
* `EASYRSA_REQ_OU`

Значения используются в качестве значений по умолчанию для всех сгенерированных запросов на сертификат, включая корневой CA. Убедитесь, что эти строки не закомментированы в файле. Никаких других изменений в `vars` не требуется.

Если определенный вами каталог `EASYRSA` не существует, создайте его сейчас и скопируйте файл `openssl-1.0.cnf` из пакета дистрибутива в ваш новый каталог. Для наших примеров мы поместили хранилище сертификатов `EASYRSA` в `/usr/local/etc/easyrsa`:

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> mkdir -p /usr/local/etc/easyrsa
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> cp openssl-1.0.cnf /usr/local/etc/easyrsa
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> cp –R x509-types /usr/local/etc/easyrsa/
```

Далее мы готовы инициализировать EasyRSA PKI:

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> ./easyrsa init-pki
```

Обратите внимание, что мы используем конфигурацию EasyRSA из `./vars`.

```
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /usr/local/etc/easyrsa/pki
```

В этом случае процесс инициализации очищает содержимое, в данном случае, каталога `pki` и создает подкаталоги `private` и `reqs`.

Вы можете иметь несколько файлов `vars` для управления несколькими ЦС, и все они могут быть вложены в один корневой каталог `EASYRSA`. Чтобы сделать это, вы должны изменить переменную `EASYRSA_PKI` для каждого ЦС.

### Создание ЦС

Подкоманда `build-ca` сначала генерирует **запрос на подпись сертификата (Certificate Signing Request - CSR)**, а затем самостоятельно подписывает этот запрос.
Чтобы создать пару сертификат/ключ корневого центра сертификации, выполните команду `build-ca`:

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> ./easyrsa build-ca
Note: using EasyRSA configuration from: ./vars
Generating a 2048 bit RSA private key
.......+++
............+++
writing new private key to '/usr/local/etceasyrsa/pki/private/ca.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
```

Вас попросят ввести информацию, которая будет включена в ваш запрос на сертификат. То, что вы собираетесь ввести, называется то, что называется отличительным именем или DN. Есть довольно много полей, но вы можете оставить некоторые пустыми; в некоторых полях будет определено значение по умолчанию, если вы введете '.', поле останется пустым.

```
-----
Common Name (eg: your user, host, or server name) [EasyRSA CA]:Mastering OpenVPN
```

---

**Подсказка**

Пробелы не должны использоваться в поле Common Name. Это вызовет проблемы в EasyRSA, а также, возможно, с CCD, общим именем в качестве имени пользователя и другими случаями. Убедитесь, что после пути есть косой черты (`/`) или некоторых подкоманд (`gen-crl` и других) не будут работать.

---

**Создание CA завершено, и теперь вы можете импортировать и подписывать сертификаты. Ваш новый файл сертификата CA для публикации находится по адресу: /usr/local/etc/easyrsa/pki/ca.crt**

При самоподписании для ограничения CA устанавливается значение true и определяются параметры использования ключа, что позволяет этому новому сертификату подписывать другие сертификаты, включая список отзыва сертификатов (CRL). Эта информация может быть проверена с помощью
утилиты командной строки openssl:

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> openssl x509 -in /usr/local/etc/easyrsa/pki/ca.crt -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            89:39:42:bb:f3:6b:a9:f6
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Mastering OpenVPN
        Validity
            Not Before: Oct 1 15:02:44 2014 GMT
            Not After : Sep 28 15:02:44 2024 GMT
        Subject: CN=Mastering OpenVPN
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (2048 bit)
                Modulus (2048 bit):
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:

69:6C:A3:85:63:61:09:DE:8F:7D:38:F7:A2:CB:1C:31:75:90:34:93
            X509v3 Authority Key Identifier:

keyid:69:6C:A3:85:63:61:09:DE:8F:7D:38:F7:A2:CB:1C:31:75:90:34:93
                DirName:/CN=Mastering OpenVPN
                serial:89:39:42:BB:F3:6B:A9:F6

            X509v3 Basic Constraints:
                CA:TRUE
            X509v3 Key Usage:
                Certificate Sign, CRL Sign
    Signature Algorithm: sha256WithRSAEncryption
        ...
```

В выводе openssl мы видим основные ограничения x509v3, а также параметры использования ключа x509v3. Теперь ваш ЦС готов начать подписывать сертификаты клиента и сервера.

### Список отзыва сертификатов

Субкоманда `gen-crl` генерирует список отзыва сертификатов. Хотя на данный момент у нас есть только сертификат CA, рекомендуется создать пустой CRL. Это позволяет вам создавать конфигурацию OpenVPN с файлом и не требует перезагрузки позже. OpenVPN будет регистрировать ошибки, если в его конфигурации указан несуществующий CRL, но файл можно заменить на лету, так как он перечитывается при каждом клиентском подключении.

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> ./easyrsa gen-crl

Note: using EasyRSA configuration from: ./vars
Using configuration from /usr/local/etc/easyrsa/openssl-1.0.cnf
Enter pass phrase for /usr/local/etc/easyrsa/pki/private/ca.key:

An updated CRL has been created.
CRL file: /usr/local/etc/easyrsa/pki/crl.pem
```

Мы можем проверить CRL, используя команду `openssl crl`:

```
ecrist@computer:~/Downloads/EasyRSA-3.0.0-rc2-> openssl crl -noout -text -in /usr/local/etc/easyrsa/pki/crl.pem
Certificate Revocation List (CRL):
        Version 2 (0x1)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: /CN=Mastering OpenVPN
        Last Update: Oct 1 15:50:46 2014 GMT
        Next Update: Mar 30 15:50:46 2015 GMT
        CRL extensions:
            X509v3 Authority Key Identifier:

keyid:69:6C:A3:85:63:61:09:DE:8F:7D:38:F7:A2:CB:1C:31:75:90:34:93
                DirName:/CN=Mastering OpenVPN
                serial:89:39:42:BB:F3:6B:A9:F6

No Revoked Certificates.
    Signature Algorithm: sha256WithRSAEncryption
        ...
```

### Серверные сертификаты

OpenVPN может использовать параметры использования ключа x509, гарантируя, что клиенты соединяются с действительными клиентскими сертификатами, и клиенты могут гарантировать, что сервер авторизован как сервер. Это предотвращает использование одного из ваших клиентских сертификатов в качестве сервера при атаке **Man-In-The-Middle (MITM)**. Без этого ограничения любой сертификат, подписанный ЦС VPN, может использоваться для олицетворения клиента или сервера. Поскольку поддельный сертификат находится в той же PKI, сам сертификат все еще действителен и будет проходить общие проверки PKI.

EasyRSA поддерживает подписывание сертификатов с помощью параметра использования ключа сервера с использованием встроенной подкоманды `build-server-full`.

```
ecrist@computer:~/EasyRSA-3.0.0-rc2-> ./easyrsa build-server-full movpn-server

Note: using EasyRSA configuration from: ./vars
Generating a 2048 bit RSA private key
............+++
..............................+++
writing new private key to '/usr/local/etc/easyrsa/pki/private/movpnserver.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from /usr/local/etc/easyrsa/openssl-1.0.cnf
Enter pass phrase for /usr/local/etc/easyrsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :PRINTABLE:'movpn-server'
Certificate is to be certified until Oct 18 19:15:31 2024 GMT (3650
days)

Write out database with 1 new entries
Data Base Updated
```

Мы можем проверить сертификат сервера с помощью команды `openssl`:

```
ecrist@computer:~/EasyRSA-3.0.0-rc2-> openssl x509 -noout -text -in /usr/local/etc/easyrsa/pki/issued/movpn-server.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1 (0x1)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Mastering OpenVPN
        Validity
            Not Before: Oct 21 19:15:31 2014 GMT
            Not After : Oct 18 19:15:31 2024 GMT
        Subject: CN=movpn-server
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:

11:AD:47:E2:1C:87:91:3B:C0:A1:53:F5:77:A7:2F:9F:0B:0F:5D:9E
            X509v3 Authority Key Identifier:

keyid:F0:AF:20:ED:6F:A7:47:0F:C7:F2:B0:EC:CF:8B:30:09:02:E4:81:A0
                DirName:/CN=Mastering OpenVPN
                serial:84:9E:B9:14:2B:62:B4:50

            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Key Usage:
                Digital Signature, Key Encipherment
    Signature Algorithm: sha256WithRSAEncryption
        ...
```

Обратите внимание на раздел использования ключа x509v3 и его идентификацию `TLS Web Server Authentication`. Это то, что текущие версии OpenVPN ищут, когда указаны типы удаленных сертификатов.

### Клиентские сертификаты

Как и сертификаты сервера, клиенты могут проходить проверку подлинности с использованием клиентских сертификатов. При использовании этого метода каждому клиенту может потребоваться уникальный сертификат. **Общее имя сертификата (Common Name - CN)** можно использовать для определения других параметров, которые должны передаваться в данном соединении с помощью сценариев `client-connect` или опции `client-config-dir`. Начиная с OpenVPN 2.3.7, все еще поддерживается `-client-cert-not-required`. Говорили об удалении этой поддержки из будущего выпуска. `client-cert-not-required` позволяет клиенту подключаться без уникального (или какого-либо) предварительно определенного сертификата, так же как веб-браузер будет подключаться к веб-серверу.

Команда `easyrsa` используется для создания сертификата клиента почти так же, как мы создали сертификат сервера:

```
ecrist@computer:~/EasyRSA-3.0.0-rc2-> ./easyrsa build-client-full
client1
Note: using EasyRSA configuration from: ./vars
Generating a 2048 bit RSA private key
...............+++
...................................................................
.......................................................+++
writing new private key to 'homeecrist/EasyRSA-3.0.0-rc2/pki/private/client1.key'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
Using configuration from homeecrist/EasyRSA-3.0.0-rc2/openssl-1.0.cnf
Enter pass phrase for homeecrist/EasyRSA-3.0.0-rc2/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName :PRINTABLE:'client1'
Certificate is to be certified until Oct 18 19:37:40 2024 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated
```

Мы можем использовать команду `openssl` для проверки параметров использования ключа:

```
ecrist@computer:~/EasyRSA-3.0.0-rc2-> openssl x509 -noout -text -in /usr/local/etc/easyrsa/pki/issued/client1.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 2 (0x2)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=Mastering OpenVPN
        Validity
            Not Before: Oct 21 19:37:40 2014 GMT
            Not After : Oct 18 19:37:40 2024 GMT
        Subject: CN=client1
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Subject Key Identifier:

47:80:59:8D:5F:63:4B:1C:21:C6:DE:C6:C0:7B:DE:6A:5D:53:F9:37
            X509v3 Authority Key Identifier:

keyid:F0:AF:20:ED:6F:A7:47:0F:C7:F2:B0:EC:CF:8B:30:09:02:E4:81:A0
                DirName:/CN=Mastering OpenVPN
                serial:84:9E:B9:14:2B:62:B4:50
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Key Usage:
                Digital Signature
    Signature Algorithm: sha256WithRSAEncryption
...
```

Как отмечено в разделе о сервере, мы рассмотрим `x509v3 Extended Key Usage`. В случае клиента мы ищем `TLS Web Client Authentication`. Опять же, это используется при проверке типа сертификата удаленного клиента.
