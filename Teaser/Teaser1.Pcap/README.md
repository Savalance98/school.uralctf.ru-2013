Вчера днём нам удалось перехватить трафик одного из наших учеников.
Говорят, что его провайдер подозревал его в хакерской активности.
Пожалуйста, помогите нам найти способ снять с него подозрения, прежде чем мы начнём собственное расследование.



Разбор:
Хм. Нам повезло и у файла есть расширение. pcap... Что же это? 
http://www.google.ru/search?q=pcap
О, google снова нас спасает и дает ссылку на Вики: http://ru.wikipedia.org/wiki/Pcap
[quote]Примером программного обеспечения, использующего библиотеку Pcap, служит программа Wireshark. [/quote]
А вот это уже интереснее. Wireshark (ранее — Ethereal) — программа-анализатор трафика. То что нужно! Находим, скачиваем, открываем с её помощью файлик.

В главном окне отображаются пакеты, пойманные в задании в виде таблицы: Номер, время от начала дампа в секундах, адрес источника, адрес назначения, протокол передачи, размер пакета, дополнительная информация. Беглым взглядом определяем, что в основном трафик протокола HTTP, так что закрадывается подозрение, что можно узнать что-то из веб общения. 
Интернет любезно подсказывает, что HTTP работает по сценарию: Запрос-Ответ, причём основными запросами являются GET и POST (отличающиеся лишь способом отправки данных). С таким знанием мы уже многое можем выудить из дампа! например, то что адрес клиента (возможно это и есть сотрудник) 192.168.185.92.
Попробуем для начала определить, куда отправлялись запросы. Для этого, для начала включим отображение DNS имен, а не адресов: View -> Name resolution -> Enable for network layer, View -> Reload.
А потом отфильтруем только http запросы, без ответов: Filter -> "ip.src == 192.168.185.92 and http"
Всё стало намного понятнее!(кстати, так бывает не всегда)

По порядку:
    1. Загружена страничка http://ructf.org/ (С запроса 11)
    2. Совершен переход на страницу http://ructf.org/2013/ (С запроса 100)
        Запросов так много, потому что прогружается не одна страница а много связанных файов к ней, в том числе различные Яндекс.Метрики, реклама и прочее.
    3. Пакетом 476 совершен запрос на загрузку http://rambler.ru
        Дальше идёт немного работы с рамблером
    4. Поиск в google.ru с запросом wireshark (видно в info) В пакете 1338.
    5. Пакет 1535 - открывает страничку http://www.wireshark.org/download.html
        Скачивать так и не стал..

Просмотрели дамп - ничего не понятно... Хотя нет. Взгляд зацепляется за POST запрос среди всех GET. Вспоминаем: POST используется для отправки данных, не отображаемых в адресной строке. Чаще всего используется для авторизации.. О! И правда!900: POST запрос на id.rampler.ru/script/auth.cgi
Кликаем два раза на пакете, он открывается в новом окне. Раскрываем содержимое пакета в самой последней секции Line-based text data (то есть передаваемый текст)
Нам повезло и мы видим заветные буквы: login=***&passw=*** (скрыто в разборе в целях интриги).
Пробуем авторизоваться на ramler'e.

В почтовом ящике множество писем и неясно что делать. Перечитываем задание. Провайдер подозревал в хакерской активности. Ищем по слову "хакер", "провайдер", о! по слову "активность" отображается интересное письмо с ключиком. 

Надо же, это и есть ответ!
