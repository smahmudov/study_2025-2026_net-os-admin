---
## Front matter
title: "Отчёт по лабораторной работе 2"
subtitle: "Настройка DHCP-сервера"
author: "Суннатилло Махмудов"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Приобретение практических навыков по установке и конфигурированию DHCP-сервера.

# Теоретические сведения

**DHCP (Dynamic Host Configuration Protocol)** — сетевой протокол, позволяющий автоматически выдавать клиентам IP-адреса и другие параметры для работы в сети. Он избавляет администратора от необходимости вручную настраивать каждый хост.

* **DHCP-сервер** — программа или служба, автоматически раздающая сетевые параметры клиентам.  
* **DHCP-клиент** — устройство или программа, запрашивающая настройки у сервера.  
* **DHCP-ретранслятор (Relay Agent)** — пересылает запросы от клиентов в другой подсети к серверу DHCP.  
* **Пул адресов (scope)** — диапазон IP-адресов, из которых сервер выдает клиентам уникальные адреса.  

## Основные параметры DHCP

* **IP-адрес** клиента;  
* **Маска подсети (subnet mask)**;  
* **Основной шлюз (default gateway)**;  
* **Адреса DNS-серверов**;  
* **Время аренды (lease time)** — срок действия выданного адреса;  
* Дополнительные параметры (WINS, доменное имя, TFTP-сервер и др.).  

1. **Discover** — клиент отправляет широковещательный запрос о наличии DHCP-сервера.  
2. **Offer** — сервер предлагает свободный IP-адрес и параметры.  
3. **Request** — клиент подтверждает выбор предложенного адреса.  
4. **Acknowledge (ACK)** — сервер закрепляет адрес за клиентом и сообщает время аренды.  


# Выполнение лабораторной работы

## Конфигурирование DHCP-сервера

1. На виртуальной машине **server** был установлен пакет **kea-dhcp4**, а также необходимые зависимости. Для сохранности конфигурации был выполнен бэкап файла:  

   cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf__$(date -I)

   ![Установка и резервное копирование конфигурации](01.png){ #fig:001 width=80% }

2. Файл конфигурации **/etc/kea/kea-dhcp4.conf** был изменён в соответствии с заданием:  
   – Установлены параметры **domain-name** и **domain-search** с использованием домена *smahmudov.net*;  
   – Определён DNS-сервер:  

   {
     "name": "domain-name-servers",
     "data": "192.168.1.1"
   }

   – Настроена подсеть с идентификатором **1**:  

   "subnet4": [
     {
       "id": 1,
       "subnet": "192.168.1.0/24",
       "pools": [ { "pool": "192.168.1.30 - 192.168.1.199" } ],
       "option-data": [
         {
           "name": "routers",
           "data": "192.168.1.1"
         }
       ]
     }
   ]

   ![Редактирование параметров domain-name и domain-search](02.png){ #fig:002 width=80% }  
   
   ![Задание конфигурации подсети](03.png){ #fig:003 width=80% }

3. Для привязки работы DHCP-сервера к интерфейсу **eth1** была внесена настройка:  

   "interfaces-config": {
     "interfaces": [ "eth1" ]
   }

   ![Привязка DHCP-сервера к интерфейсу](04.png){ #fig:004 width=80% }

4. В прямой зоне DNS **/var/named/master/fz/smahmudov.net** добавлена запись для DHCP-сервера:  

   dhcp     A   192.168.1.1

   ![Файл прямой зоны](05.png){ #fig:005 width=80% }

5. В обратной зоне DNS **/var/named/master/rz/192.168.1** добавлена PTR-запись для DHCP-сервера:  

   1   PTR dhcp.smahmudov.net.

   ![Файл обратной зоны](06.png){ #fig:006 width=80% }

6. После редактирования конфигурационных файлов был перезапущен сервис **named**:  

   systemctl restart named

7. Проверка доступности DHCP-сервера по имени подтвердила правильность настроек:  

   ping dhcp.smahmudov.net

   ![Проверка доступности DHCP-сервера по имени](07.png){ #fig:007 width=80% }

8. Для разрешения работы DHCP-сервиса в межсетевом экране были внесены изменения:  

   firewall-cmd --list-services  
   firewall-cmd --get-services  
   firewall-cmd --add-service=dhcp  
   firewall-cmd --add-service=dhcp --permanent  

   ![Разрешение DHCP в firewall](08.png){ #fig:008 width=80% }

9. Для восстановления контекста безопасности в **SELinux** были выполнены команды:  

    restorecon -vR /etc  
    restorecon -vR /var/named  
    restorecon -vR /var/lib/kea/  

10. В дополнительном терминале был запущен мониторинг происходящих в системе процессов в реальном времени:  

    tail -f /var/log/messages  

11. В основном рабочем терминале был запущен сервис **kea-dhcp4**:  

    systemctl start kea-dhcp4.service  

12. После успешного запуска DHCP-сервера, не выключая виртуальной машины **server** и не прерывая мониторинга процессов, был выполнен переход к анализу работы DHCP-сервера на клиенте.  

## Анализ работы DHCP-сервера

1. Перед запуском виртуальной машины **client** был создан скрипт **01-routing.sh** в каталоге *vagrant/provision/client*. Он изменяет настройки **NetworkManager**, чтобы весь трафик клиента проходил через интерфейс **eth1**.  

2. В файл **Vagrantfile** был добавлен вызов данного скрипта, после чего виртуальная машина **client** была запущена командой:  

   make client-provision  

   (для Windows — *vagrant up client --provision*).

3. После загрузки клиентской машины сервер **server** выдал клиенту IP-адрес из заданного диапазона. Это событие зафиксировано в логах и в файле **/var/lib/kea/kea-leases4.csv**.  

   ![Список аренд DHCP](10.png){ #fig:009 width=80% }

4. На виртуальной машине **client** была выполнена команда **ifconfig**, что подтвердило получение IP-адреса **192.168.1.30** на интерфейсе **eth1**.  

   ![Результат работы ifconfig](09.png){ #fig:010 width=80% }

   **Комментарий к выводу:**  
   – **eth1**: клиент получил адрес `192.168.1.30` с маской `255.255.255.0` и широковещательным адресом `192.168.1.255`. Также отображён MAC-адрес сетевой карты клиента `08:00:27:d2:8d:de`.  
   – **lo**: интерфейс *loopback* имеет адрес `127.0.0.1`, используется для локального взаимодействия внутри машины.  

5. На сервере в файле **kea-leases4.csv** отображается информация о выданных адресах.  

   **Пример записи:**  
   ```
   192.168.1.30,08:00:27:d2:8d:de,01:08:00:27:d2:8d:de,3600,1757837446,1,0,0,client,,0,,0
   ```

   **Пояснение к полям:**  
   – `192.168.1.30` — IP-адрес, выданный клиенту.  
   – `08:00:27:d2:8d:de` — MAC-адрес клиента.  
   – `01:08:00:27:d2:8d:de` — client-id.  
   – `3600` — время аренды в секундах (1 час).  
   – `1757837446` — метка времени окончания аренды.  
   – `1` — идентификатор подсети, из которой был выдан адрес.  
   – `client` — hostname клиента.  
   – остальные поля зарезервированы и в текущем случае пустые.  

## Настройка обновления DNS-зоны

1. На сервере с **Bind9** был создан ключ для динамических обновлений:  

   mkdir -p /etc/named/keys  
   tsig-keygen -a HMAC-SHA512 DHCP_UPDATER > /etc/named/keys/dhcp_updater.key  

   ![Создание ключа TSIG](11.png){ #fig:011 width=80% }

2. Ключ был подключён в конфигурацию **named** через файл */etc/named.conf*.  
   В зоны были внесены правила обновления:  

   zone "smahmudov.net" IN {  
   type master;  
   file "master/fz/smahmudov.net";  
   update-policy {  
   grant DHCP_UPDATER wildcard *.smahmudov.net A DHCID;  
   };  
   };  

   zone "1.168.192.in-addr.arpa" IN {  
   type master;  
   file "master/rz/192.168.1";  
   update-policy {  
   grant DHCP_UPDATER wildcard *.1.168.192.in-addr.arpa PTR DHCID;  
   };  
   };  

   ![Разрешение обновлений зоны](12.png){ #fig:012 width=80% }

3. После проверки синтаксиса командой **named-checkconf** был перезапущен сервис **named**:  

   systemctl restart named  

4. Для взаимодействия с Kea DHCP был создан файл ключа **/etc/kea/tsig-keys.json**, в который был перенесён ключ в формате JSON:  

   {
     "tsig-keys": [
       {
         "name": "DHCP_UPDATER",
         "algorithm": "hmac-sha512",
         "secret": "…"
       }
     ]
   }

   ![Файл tsig-keys.json](13.png){ #fig:013 width=80% }

5. Настройка динамического обновления была выполнена в файле **/etc/kea/kea-dhcp-ddns.conf**. В нём определены forward- и reverse-зоны с указанием сервера и ключа:  

   ![Файл kea-dhcp-ddns.conf](14.png){ #fig:014 width=80% }

6. После проверки конфигурации сервис **kea-dhcp-ddns** был включён и запущен:  

   systemctl enable --now kea-dhcp-ddns.service  
   systemctl status kea-dhcp-ddns.service  

   ![Запуск kea-dhcp-ddns.service](15.png){ #fig:015 width=80% }

7. В основной конфигурации DHCP (**/etc/kea/kea-dhcp4.conf**) была включена поддержка обновлений DNS:  

   "dhcp-ddns": {  
     "enable-updates": true  
   },  
   "ddns-qualifying-suffix": "smahmudov.net",  
   "ddns-override-client-update": true  

   ![Изменения в конфигурации DHCP](16.png){ #fig:016 width=80% }

8. Сервис **kea-dhcp4** был перезапущен и проверен:  

   systemctl restart kea-dhcp4.service  
   systemctl status kea-dhcp4.service  

   ![Перезапуск kea-dhcp4](17.png){ #fig:017 width=80% }

9. На клиентской машине было выполнено переподключение интерфейса **eth1** для обновления IP-адреса:  

   nmcli connection down eth1  
   nmcli connection up eth1  
   
## Анализ работы DHCP-сервера после настройки обновления DNS-зоны

В результате в DNS появилась динамически добавленная запись. Проверка с помощью утилиты **dig** подтвердила корректность работы DDNS:  

    dig @192.168.1.1 client.smahmudov.net  

    ![Проверка DNS-записи клиента](18.png){ #fig:018 width=80% }

## Внесение изменений в настройки внутреннего окружения виртуальной машины

Для автоматизации процесса настройки DHCP и DDNS был подготовлен скрипт, который выполняет следующие действия:  

![Скрипт настройки внутреннего окружения](19.png){ #fig:019 width=80% }


# Вывод

В ходе лабораторной работы был развернут и настроен DHCP-сервер **Kea** с привязкой к сетевому интерфейсу и интеграцией с системой имен **DNS**. Реализовано автоматическое назначение IP-адресов клиентам, настройка прямой и обратной зон, а также динамическое обновление DNS-записей через DDNS. Проверка с помощью утилит **ifconfig**, **ping** и **dig** подтвердила корректную работу DHCP и DNS-сервисов.  

# Контрольные вопросы

1. **В каких файлах хранятся настройки сетевых подключений?**  
   Настройки сетевых подключений в Linux обычно хранятся в каталоге **/etc/sysconfig/network-scripts/** (например, файлы *ifcfg-eth0*, *ifcfg-eth1*).  
   В современных системах с **NetworkManager** конфигурация хранится в каталоге **/etc/NetworkManager/system-connections/** в виде файлов формата *.nmconnection*.  

2. **За что отвечает протокол DHCP?**  
   DHCP (Dynamic Host Configuration Protocol) — это протокол динамической конфигурации хостов, который автоматизирует назначение IP-адресов, маски подсети, шлюза и DNS-серверов клиентским устройствам в сети.  

3. **Поясните принцип работы протокола DHCP. Какими сообщениями обмениваются клиент и сервер, используя протокол DHCP?**  
   Принцип работы DHCP основан на механизме аренды IP-адресов. Клиент и сервер обмениваются следующими сообщениями:  
   – **DHCPDISCOVER** — клиент ищет сервер;  
   – **DHCPOFFER** — сервер предлагает адрес и параметры;  
   – **DHCPREQUEST** — клиент подтверждает выбор;  
   – **DHCPACK** — сервер окончательно закрепляет за клиентом адрес.  

4. **В каких файлах обычно находятся настройки DHCP-сервера? За что отвечает каждый из файлов?**  
   Для DHCP-сервера **Kea** используются следующие файлы:  
   – */etc/kea/kea-dhcp4.conf* — основная конфигурация DHCPv4 (подсети, параметры аренды, опции);  
   – */etc/kea/kea-dhcp-ddns.conf* — настройки динамических обновлений DNS (DDNS);  
   – */etc/kea/tsig-keys.json* — ключи для аутентификации при обмене с DNS-сервером;  
   – */var/lib/kea/kea-leases4.csv* — база данных с информацией о выданных адресах.  

5. **Что такое DDNS? Для чего применяется DDNS?**  
   DDNS (Dynamic DNS) — это механизм динамического обновления DNS-записей. Применяется для того, чтобы при выдаче клиенту нового IP-адреса DHCP-сервер автоматически добавлял (или обновлял) соответствующую A- и PTR-записи в DNS-зоне. Это обеспечивает корректное разрешение имён в сети.  

6. **Какую информацию можно получить, используя утилиту ifconfig? Приведите примеры с использованием различных опций.**  
   Утилита **ifconfig** позволяет просмотреть и настроить сетевые интерфейсы.  
   Примеры:  
   – `ifconfig` — отображение всех активных интерфейсов;  
   – `ifconfig eth1` — подробная информация об интерфейсе eth1 (IP, MAC, статистика пакетов);  
   – `ifconfig eth1 down` / `ifconfig eth1 up` — отключение и включение интерфейса.  

7. **Какую информацию можно получить, используя утилиту ping? Приведите примеры с использованием различных опций.**  
   Утилита **ping** проверяет доступность узлов и измеряет время отклика.  
   Примеры:  
   – `ping 192.168.1.1` — проверка доступности маршрутизатора по IP;  
   – `ping dhcp.smahmudov.net` — проверка доступности узла по имени (через DNS);  
   – `ping -c 5 8.8.8.8` — отправка фиксированного количества пакетов (5);  
   – `ping -s 1024 8.8.8.8` — отправка пакетов увеличенного размера (1024 байта).  
	
# Список литературы

1. Barr D. Common DNS Operational and Configuration Errors: RFC / RFC Editor. — 02/1996. — DOI: 10.17487/rfc1912.

2. Droms R. Dynamic Host Configuration Protocol: RFC / RFC Editor. — 03/1997. — P. 1–45. — DOI: 10.17487/rfc2131.

3. Dynamic Updates in the Domain Name System (DNS UPDATE), RFC 2136: RFC / P. Vixie, S. Thomson, Y. Rekhter, J. Bound; RFC Editor. — 04/1997. — DOI: 10.17487/RFC2136.
