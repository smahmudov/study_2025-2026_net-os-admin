---
## Front matter
title: "Отчёт по лабораторной работе 9"
subtitle: "Настройка POP3/IMAP сервера"
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

Приобретение практических навыков по установке и простейшему конфигурированию POP3/IMAP-сервера.

# Теоретические сведения

Электронная почта (E-mail) является одним из основных сервисов сетей передачи данных. Она обеспечивает обмен текстовыми и мультимедийными сообщениями между пользователями по сетевым протоколам, определяющим порядок передачи и получения писем.

## Основные почтовые протоколы

1. **SMTP (Simple Mail Transfer Protocol)**  
   Используется для передачи сообщений от клиента к серверу и между серверами. Работает по принципу "push" — инициатором передачи всегда выступает отправитель.  
   Основной порт — **25**, а также **587** (STARTTLS) и **465** (SSL).

2. **POP3 (Post Office Protocol version 3)**  
   Применяется для получения почты с сервера. После загрузки письма на клиент оно, как правило, удаляется с сервера. POP3 прост в реализации и подходит для однопользовательской работы.  
   Основные порты — **110** (без шифрования) и **995** (SSL).

3. **IMAP (Internet Message Access Protocol)**  
   Позволяет пользователю управлять письмами, хранящимися на сервере, без их полного скачивания. Поддерживает синхронизацию между несколькими клиентами, сортировку, поиск и создание папок.  
   Основные порты — **143** и **993** (SSL).

## Почтовые серверы

1. **Postfix**  
   Почтовый транспортный агент (MTA), обеспечивающий приём, маршрутизацию и отправку сообщений по протоколу SMTP. Он отвечает за доставку писем к другим серверам или локально пользователям.

2. **Dovecot**  
   Почтовый сервер (MDA — Mail Delivery Agent и IMAP/POP3 сервер), обеспечивающий доступ пользователей к почтовым ящикам. Dovecot выполняет аутентификацию, управление почтовыми каталогами и взаимодействует с Postfix при доставке писем.

## Форматы хранения почты

– **mbox** — все письма хранятся в одном файле.  
– **Maildir** — каждое письмо сохраняется в отдельном файле, что повышает надёжность и упрощает доступ при большом объёме сообщений.

## Аутентификация пользователей

Dovecot поддерживает различные механизмы аутентификации:
- **plain** — простой пароль в открытом виде (часто используется с TLS/SSL);
- **login** — классическая схема авторизации SMTP;
- **cram-md5**, **digest-md5** — методы с хешированием пароля;
- **PAM**, **passwd**, **SQL**, **LDAP** — системы проверки подлинности на основе локальных или сетевых баз данных.

## Взаимодействие Postfix и Dovecot

Postfix отвечает за доставку сообщений в почтовый ящик пользователя, после чего Dovecot предоставляет пользователю доступ к ним по IMAP или POP3.  
Такое разделение функций обеспечивает безопасность, масштабируемость и гибкость настройки почтовой системы.

# Выполнение лабораторной работы

## Настройка почтового сервера Dovecot

### Установка Dovecot

1. На виртуальной машине **server** был выполнен вход под пользователем и произведён переход в режим суперпользователя:  
   sudo -i

2. Установлены необходимые для работы пакеты **dovecot** и **telnet**:  
   dnf -y install dovecot telnet  

### Настройка Dovecot

1. В конфигурационном файле **/etc/dovecot/dovecot.conf** был указан список поддерживаемых почтовых протоколов:  
   protocols = imap pop3  

   ![Редактирование dovecot.conf — задание протоколов IMAP и POP3](01.png){ #fig:001 width=80% }

2. В файле **/etc/dovecot/conf.d/10-auth.conf** проверено, что используется метод аутентификации **plain**:  
   auth_mechanisms = plain  

   ![Проверка параметра auth_mechanisms в 10-auth.conf](02.png){ #fig:002 width=80% }

3. В файле **/etc/dovecot/conf.d/auth-system.conf.ext** определено использование PAM и системного файла паролей:  
   passdb {  
       driver = pam  
   }  

   userdb {  
       driver = passwd  
   }  

   ![Настройка аутентификации через PAM и passwd](03.png){ #fig:003 width=80% }

4. В файле **/etc/dovecot/conf.d/10-mail.conf** указано месторасположение пользовательских почтовых ящиков:  
   mail_location = maildir:~/Maildir  

   ![Указание каталога хранения писем Maildir](04.png){ #fig:004 width=80% }

### Настройка Postfix и системы

1. В системе **Postfix** был задан каталог доставки почты:  
   postconf -e 'home_mailbox = Maildir/'

2. Внесены изменения в конфигурацию межсетевого экрана, чтобы разрешить работу служб **POP3/POP3S** и **IMAP/IMAPS**:  
   firewall-cmd --add-service=pop3 --permanent  
   firewall-cmd --add-service=pop3s --permanent  
   firewall-cmd --add-service=imap --permanent  
   firewall-cmd --add-service=imaps --permanent  
   firewall-cmd --reload  
   firewall-cmd --list-services  

   ![Настройка межсетевого экрана и разрешение служб IMAP и POP3](05.png){ #fig:005 width=80% }

3. Восстановлен контекст безопасности SELinux:  
   restorecon -vR /etc  

4. Почтовые службы **Postfix** и **Dovecot** были перезапущены и добавлены в автозагрузку:  
   systemctl restart postfix  
   systemctl enable dovecot  
   systemctl start dovecot  

## Проверка работы Dovecot

1. На дополнительном терминале виртуальной машины **server** был запущен мониторинг работы почтовой службы:  
   tail -f /var/log/maillog  

   ![Мониторинг журнала maillog](06.png){ #fig:006 width=80% }

2. На виртуальной машине **client** был установлен и настроен почтовый клиент **Evolution**.  
   В параметрах подключения указаны:  
   – **IMAP-сервер:** mail.smahmu­dov.net  
   – **SMTP-сервер:** mail.smahmu­dov.net  
   – **Пользователь:** smahmu­dov  
   – **Тип безопасности:** STARTTLS  

   ![Настройка учётной записи в почтовом клиенте Evolution](07.png){ #fig:007 width=80% }

3. После настройки клиента было выполнено тестовое отправление письма самому себе.  
   В папке **Inbox** успешно отобразилось входящее сообщение с темой *test1*.  

   ![Получение тестового письма в почтовом клиенте Evolution](08.png){ #fig:008 width=80% }
   
   ![Мониторинг журнала maillog](09.png){ #fig:009 width=80% }

4. На терминале сервера для просмотра имеющейся почты использовалась команда:  
   MAIL=~/Maildir mail  

   Было получено входящее сообщение с темой **test1**, отправленное пользователем **smahmu­dov@smahmu­dov.net**.  

   ![Просмотр входящего письма с помощью mail](10.png){ #fig:010 width=80% }

5. Для проверки работы POP3-протокола была выполнена сессия подключения через **Telnet**.  
   После входа и выполнения команд `list`, `retr 1`, `dele 2` и `quit` подтверждена возможность чтения и удаления писем на сервере.  

   ![Проверка POP3 через Telnet](11.png){ #fig:011 width=80% }

## Внесение изменений в настройки внутреннего окружения виртуальной машины

1. На сервере был создан каталог для хранения конфигурационных файлов почтовой системы:  
   /vagrant/provision/server/mail/etc/dovecot/conf.d  

2. В каталог были скопированы рабочие конфигурационные файлы Dovecot:  
   – dovecot.conf  
   – 10-auth.conf  
   – auth-system.conf.ext  
   – 10-mail.conf  

   ![Копирование конфигурационных файлов Dovecot в каталог Vagrant](12.png){ #fig:0012 width=80% }

3. В файл **/vagrant/provision/server/mail.sh** необходимо добавить команды установки Dovecot и Telnet, а также настройки межсетевого экрана, чтобы автоматизировать развёртывание почтового сервера в виртуальной среде.

# Вывод

В ходе лабораторной работы был установлен и настроен почтовый сервер **Dovecot**, обеспечивающий работу по протоколам **IMAP** и **POP3**, а также почтовый транспортный агент **Postfix** для отправки сообщений по протоколу **SMTP**.  
Были внесены изменения в конфигурационные файлы Dovecot, настроены методы аутентификации и расположение почтовых ящиков пользователей.  
С помощью клиента **Evolution** и утилит командной строки (mail, telnet) проведена проверка отправки и получения писем, подтверждающая корректную работу почтовой системы.  
В результате сервер успешно выполняет функции приёма, хранения и доставки электронной почты.


# Контрольные вопросы

1. **За что отвечает протокол SMTP?**  
   SMTP (Simple Mail Transfer Protocol) — это протокол передачи электронной почты, используемый для отправки писем от клиента на почтовый сервер и для пересылки сообщений между серверами. Он работает, как правило, через порт **25**, а также может использовать **587** (с шифрованием STARTTLS) и **465** (SSL).  

2. **За что отвечает протокол IMAP?**  
   IMAP (Internet Message Access Protocol) — протокол для доступа к почтовым ящикам на сервере. Он позволяет пользователю работать с письмами удалённо: просматривать, сортировать, удалять, создавать папки, не загружая письма полностью на клиент. Обычно использует порт **143** или **993** (SSL).  

3. **За что отвечает протокол POP3?**  
   POP3 (Post Office Protocol, версия 3) — протокол для получения почты с сервера. В отличие от IMAP, письма обычно скачиваются на клиентское устройство и удаляются с сервера. Использует порт **110** или **995** (SSL).  

4. **В чём назначение Dovecot?**  
   Dovecot — это сервер, обеспечивающий доступ к электронной почте через протоколы **IMAP** и **POP3**. Он выполняет аутентификацию пользователей, управление почтовыми ящиками, а также обеспечивает безопасность доступа к почтовым данным.  

5. **В каких файлах обычно находятся настройки работы Dovecot? За что отвечает каждый из файлов?**  
   Основные конфигурационные файлы Dovecot находятся в каталоге **/etc/dovecot/**:  
   – **dovecot.conf** — главный файл конфигурации, определяющий глобальные параметры и используемые модули.  
   – **conf.d/10-auth.conf** — настройки аутентификации пользователей (методы входа, разрешённые механизмы).  
   – **conf.d/auth-system.conf.ext** — описание используемых систем аутентификации (например, PAM или passwd).  
   – **conf.d/10-mail.conf** — определяет расположение почтовых ящиков пользователей (Maildir или mbox).  
   – **conf.d/10-logging.conf** — настройки логирования.  
   – **conf.d/10-master.conf** — управление службами Dovecot и их портами.  

6. **В чём назначение Postfix?**  
   Postfix — это почтовый транспортный агент (MTA), отвечающий за приём, маршрутизацию и доставку писем. Он принимает письма от клиентов через SMTP, передаёт их другим серверам или локально доставляет в почтовые ящики пользователей.  

7. **Какие методы аутентификации пользователей можно использовать в Dovecot и в чём их отличие?**  
   В Dovecot можно использовать несколько методов аутентификации:  
   – **PAM** — аутентификация через системные учётные записи (использует /etc/shadow).  
   – **passwd** — проверка пользователей по локальному файлу /etc/passwd.  
   – **static** — статическая конфигурация для тестирования, без реальных пользователей.  
   – **SQL / LDAP** — аутентификация через базы данных или каталоги пользователей.  
   Отличие состоит в источнике данных: PAM и passwd используют системные учётные записи, SQL/LDAP — внешние источники.  

8. **Пример заголовка письма с пояснением его полей:**  

```
From: smahmu­dov <smahmu­dov@smahmu­dov.net>
To: smahmu­dov@smahmu­dov.net
Subject: test1
Date: Fri, 10 Oct 2025 05:29:18 +0000
Message-ID: <aa4c5c6206869ebd35e05dc2852fc662ecc1f7cb.camel@smahmu­dov.net>
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
```

**Пояснение полей:**  
– *From* — отправитель письма.  
– *To* — получатель.  
– *Subject* — тема письма.  
– *Date* — дата и время отправки.  
– *Message-ID* — уникальный идентификатор письма.  
– *MIME-Version* и *Content-Type* — формат содержимого письма.  

9. **Примеры использования команд для работы с почтовыми протоколами через терминал:**  
Подключение к серверу по **POP3**:  
```
telnet mail.user.net 110
user smahmu­dov
pass пароль
list
retr 1
dele 2
quit
```

Эти команды позволяют войти на почтовый сервер, получить список сообщений, просмотреть содержимое письма, удалить его и завершить сеанс.  

10. **Примеры по работе с doveadm:**  
– Просмотр списка почтовых ящиков пользователя:  
  doveadm mailbox list -u smahmu­dov  
– Проверка состояния службы Dovecot:  
  doveadm service status  
– Удаление всех писем из определённого ящика:  
  doveadm expunge -u smahmu­dov mailbox INBOX all  
– Получение статистики по пользователю:  
  doveadm who -1  
	
# Список литературы

1. Dovecot Documentation. — URL: https : / / dovecot . org / documentation (visited on 09/13/2021).
2. Postfix Documentation. — URL: http://www.postfix.org/documentation.html (visited on 09/13/2021)