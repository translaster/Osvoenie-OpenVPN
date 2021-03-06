# Содержание книги

[Глава 1. Введение в OpenVPN](chapter-01.md)

* [Что такое VPN](chapter-01.md#что-такое-vpn)
* [Типы VPN](chapter-01.md#типы-vpn)
  - [PPTP](chapter-01.md#pptp)
  - [IPSec](chapter-01.md#ipsec)
  - [VPN на основе SSL](chapter-01.md#vpn-на-основе-ssl)
  - [OpenVPN](chapter-01.md#openvpn)
* [Сравнение VPN](chapter-01.md#сравнение-vpn)
  - [Преимущества и недостатки PPTP](chapter-01.md#преимущества-и-недостатки-pptp)
  - [Преимущества и недостатки IPSec](chapter-01.md#преимущества-и-недостатки-ipsec)
  - [Преимущества и недостатки VPN на основе SSL](chapter-01.md#преимущества-и-недостатки-vpn-на-основе-ssl)
  - [Преимущества и недостатки OpenVPN](chapter-01.md#преимущества-и-недостатки-openvpn)
  - [История OpenVPN](chapter-01.md#история-openvpn)
* [Пакеты OpenVPN](chapter-01.md#пакеты-openvpn)
  - [Версия с открытым исходным кодом (сообщество)](chapter-01.md#версия-с-открытым-исходным-кодом-сообщество)
  - [Закрытые исходники (коммерческий) Access Server](chapter-01.md#закрытые-исходники-коммерческий-access-server)
  - [Мобильная платформа (смешанная) OpenVPN/OpenVPN Connect](chapter-01.md#мобильная-платформа-смешанная-openvpnopenvpn-connect)
  - [Другие платформы](chapter-01.md#другие-платформы)
* [Внутренности OpenVPN](chapter-01.md#внутренности-openvpn)
  - [Драйвер tun/tap](chapter-01.md#драйвер-tuntap)
  - [Режимы UDP и TCP](chapter-01.md#режимы-udp-и-tcp)
  - [Протокол шифрования](chapter-01.md#протокол-шифрования)
  - [Каналы управления передачи данных](chapter-01.md#каналы-управления-и-передачи-данных)
  - [Алгоритмы шифрования и хеширования](chapter-01.md#алгоритмы-шифрования-и-хеширования)
  - [OpenSSL против PolarSSL](chapter-01.md#openssl-против-polarssl)
* [Резюме](chapter-01.md#резюме)

[Глава 2. Режим точка-точка](chapter-02.md)

* [Плюсы и минусы ключевого режима](chapter-02.md#плюсы-и-минусы-ключевого-режима)
  - [Первый пример](chapter-02.md#первый-пример)
* [Протокол TCP и разные порты](chapter-02.md#протокол-tcp-и-разные-порты)
  - [Режим tap](chapter-02.md#режим-tap)
  - [Топология подсети](chapter-02.md#топология-подсети)
  - [Туннель открытого текста](chapter-02.md#туннель-открытого-текста)
* [Секретные ключи OpenVPN](chapter-02.md#секретные-ключи-openvpn)
  - [Использование нескольких ключей](chapter-02.md#использование-нескольких-ключей)
  - [Использование разных алгоритмов шифрования и аутентификации](chapter-02.md#использование-разных-алгоритмов-шифрования-и-аутентификации)
* [Маршрутизация](chapter-02.md#маршрутизация)
  - [Конфигурационные файлы и командная строка](chapter-02.md#конфигурационные-файлы-и-командная-строка)
* [Полная настройка](chapter-02.md#полная-настройка)
  - [Расширенная настройка без-IP](chapter-02.md#расширенная-настройка-без-ip)
* [Трехсторонняя маршрутизация](chapter-02.md#трехсторонняя-маршрузиация)
  - [Маршрут, net_gateway, vpn_gateway и метрики](chapter-02.md#маршрут-net_gateway-vpn_gateway-и-метрики)
  - [Мостовой tap-переходник на обоих концах](chapter-02.md#мостовой-tap-переходник-на-обоих-концах)
  - [Удаление мостов](chapter-02.md#удаление-мостов)
* [Совмещение режима точка-точка с сертификатами](chapter-02.md#совмещение-режима-точка-точка-с-сертификатами)
* [Резюме](chapter-02.md#резюме)

[Глава 3. PKI и сертификаты](chapter-03.md)

* [Обзор PKI](chapter-03.md#обзор-pki)
  - [PKI с использованием EasyRSA](chapter-03.md#pki-с-использованием-easyrsa)
  - [Создание CA](chapter-03.md#создание-ca)
  - [Список отзыва сертификатов](chapter-03.md#список-отзыва-сертификатов)
  - [Серверные сертификаты](chapter-03.md#серверные-сертификаты)
  - [Клиентские сертификаты](chapter-03.md#клиентские-сертификаты)
  - [PKI с использованием ssl-admin](chapter-03.md#pki-с-использованием-ssl-admin)
* [Серверные сертификаты OpenVPN](chapter-03.md#серверные-сертификаты-openvpn)
* [Клиентские сертификаты OpenVPN](chapter-03.md#клиентские-сертификаты-openvpn)
* [Другие преимущества](chapter-03.md#другие-преимущества)
* [Несколько CA и CRL](chapter-03.md#несколько-ca-и-crl)
* [Дополнительная безопасность - аппаратные токены, смарт-карты и PKCS#11](chapter-03.md#дополнительная-безопасность---аппаратные-токены-смарт-карты-и-pkcs-11)
  - [Исходная информация](chapter-03.md#исходная-информация)
  - [Поддерживаемые платформы](chapter-03.md#поддерживаемые-платформы)
  - [Инициализация аппаратного токена](chapter-03.md#инициализация-аппаратного-токена)
  - [Генерация пары сертификат/приватный ключ](chapter-03.md#генерация-пары-сертификатприватный-ключ)
  - [Генерация приватного ключа на токене](chapter-03.md#генерация-приватного-ключа-на-токене)
  - [Генерация запроса на сертификат](chapter-03.md#генерация-запроса-на-сертификат)
  - [Запись сертификата X.509 в токен](chapter-03.md#запись-сертификата-x509-в-токен)
  - [Получение идентификатора аппаратного токена](chapter-03.md#получение-идентификатора-аппаратного-токена)
  - [Использование аппаратного токена с OpenVPN](chapter-03.md#использование-аппаратного-токена-с-openvpn)
* [Резюме](chapter-03.md#резюме)

[Глава 4. Режим клиент/сервер с tun-устройствами](chapter-04.md)
* [Понимание режима клиент-сервер](chapter-04.md#понимание-режима-клиент-сервер)
* [Настройка инфраструктуры открытых ключей](chapter-04.md#настройка-инфраструктуры-открытых-ключей)
* [Начальная настройка режима клиент-сервер](chapter-04.md#начальная-настройка-режима-клиент-сервер)
  - [Подробное объяснение файлов конфигурации](chapter-04.md#подробное-объяснение-файлов-конфигурации)
* [Topology subnets против topology net30](chapter-04.md#topologu-subnet-против-topology-net30)
* [Добавление повышенной безопасности](chapter-04.md#добавление-повышенной-безопасности)
  - [Использование ключей tls-auth](chapter-04.md#использование-ключей-tls-auth)
  - [Генерация ключа tls-auth](chapter-04.md#генерация-ключа-tls-auth)
  - Проверка атрибутов использования ключа сертификата
* Основные конфигурационные файлы производственного уровня
  - Конфигурация на основе TCP
  - Конфигурационные файлы для Windows
* Маршрутизация и маршрутизация на стороне сервера
  - Специальные параметры для вариантов маршрута
  - [Маскарадинг](chapter-04.md#маскарадинг)
* Перенаправление шлюза по умолчанию
* Специфичная для клиента конфигурация - файлы CCD
  - Как определить, правильно ли обрабатывается файл CCD
  - CCD файлы и топология net30
* [Клиентская маршрутизация](chapter-04.md#клиентская-маршрутизация)
  - Подробное объяснение конфигурации client-config-dir
  - Трафик клиент-клиент
* [Файл состояния OpenVPN](chapter-04.md#файл-состояния-openvpn)
  - Надежное отслеживание соединения в режиме UDP
* Интерфейс управления OpenVPN
* Пересмотр ключа сеанса
  - Замечание об устройствах PKCS#11
* Использование IPv6
  - Защищенный трафик IPv6
  - Использование IPv6 в качестве транзита
* Расширенные параметры конфигурации
  - ARP-прокси
    - Как работает Proxy ARP?
  - Назначение публичных IP-адресов клиентам
* [Резюме](chapter-04.md#резюме)

[Глава 5. Расширенные сценарии развертывания в туннельном режиме](chapter-05.md)

[Глава 6. "Режим клиент/сервер" с tap-устройством](chapter-06.md)

[Глава 7. Сценарии и плагины](chapter-07.md)

[Глава 8. Использование OpenVPN на мобильных устройствах и домашних маршрутизаторах](chapter-08.md)

[Глава 9. Устранение неполадок и настройка](chapter-09.md)

[Глава 10. Будущие направления](chapter-10.md)

Index'а не будет
