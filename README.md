Wget GUI Light
=========

![license MIT](http://gitshields.com/v2/text/license/MIT/green.png) &nbsp; ![license MIT](http://gitshields.com/v2/text/os/linux/orange.png) &nbsp; ![php 4,5](http://gitshields.com/v2/text/php/4,5/blue.png) &nbsp; ![version](http://gitshields.com/v2/text/version/0.1.1/lightgrey.png)

Web-интерфейс для [wget] - программы загрузки файлов по сети. Построен на Ajax+css3 с клиентской стороны, и **php** с серверной. Требования к системе, на которой разворачивается данное решение:

 * ***nix** (по крайней мере писалось именно под эту платформу, для запуска под другой потребуются рабочие порты wget и ps);
 * **bash** (проверена работа на версиях 4.1.11(2) и 4.2.24(1))
 * **php5.x** (скорее всего работать будет и на php4.x, по к моменту публикации это протестировано не было);
 * Браузер с поддержкой **Javascript** (и очень желательно — с поддержкой css3).

Скриншот интерфейса:
![screenshot](http://habrastorage.org/files/35b/291/c45/35b291c45ba4476ebf0517416efc01d2.png)


Особенности и настройки серверной части
----
Как уже было сказано выше — в роли серверной части выступает скрипт, написанный на php. Он выполняет следующие задачи:

 - Получение информации о запущенных задачах;
 - Отмена запущенных задач;
 - Добавление новых задач;
 - Тестирование серверной части;
 - Возвращение результатов в `JSON` формате.

Для своей работы ему требуются `bash`, `ps`, `wget` и `rm`. Для получения значения состояния закачки (на сколько процентов завершена) используется следующий алгоритм:

 - Задачи `wget` запускаются с флагом "`--progress=bar:force`";
 - Вывод лога загрузки производится в файл, установленный в параметре "`--output-file=FILE`";
 - При запросе состояния задач с помощью "`ps -ax`" получаем путь к файлу, установленный в "`--output-file=FILE`";
 - Читаем крайнюю строку этого файла, получая регуляркой из него искомое значение.

Путь для временных лог-файлов устанавливается в строке "`define('tmp_path', '/tmp');`".

Путь до директории, в которую будет происходить сохранение всех загружаемых файлов устанавливается в строке "`define('download_path', BASEPATH.'/downloads');`".

Доступно удобное указание путей до `ps`, `wget` и `rm`. Для этого достаточно убрать комментарий вначале строки и указать свой путь, например: "`define('wget', '/usr/bin/wget');`".

Количество одновременных задач задается в строке `define('WgetOnetimeLimit', 10);`. Если данная функция не нужна - закомментируй эту строку.

Возможна установка ограничения на скорость закачки из секции настроек. За это отвечает строка "`define('wget_download_limit', '1024');`". Если оставить её закомментированной — никакого ограничения не будет.

Директория для логов задается в строке `define('log_path', BASEPATH.'/log');`,  а имена файлов, в которые сообщения пишутся - установлены в самом начале класса `log` (строка ~115). По умолчанию все сообщения пишутся в один файл `wgetgui.log`.

Доступна так же хистория задач. Включается снятием комментария со строки `define('history', BASEPATH.'/log/history.log');`. Так же отображется в GUI (в выдвижном меню). Количество отображаемых задач задается в ~637 строке (`$itemsCount = 10;`).

Режим отладки включается в строке `define('DebugMode', true);` и добавляет подробный вывод в лог-файл.

Для определения в списке задачи запущенной через GUI от любой другой используется определенный флаг, уникальный для GUI. Он установлен в строке "`define('wget_secret_flag', '--max-redirect=4321');`" и его менять без необходимости не надо. Кстати, если хотите чтоб другие ваши задачи, запущенные из терминала отображались в веб-интерыейсе — достаточно как раз этот параметр к ним и добавить. Но не забывайте, что есть ещё и некоторые другие параметры, не менее обязательные (смотри сорцы `function addWgetTask($url)`).

Постарался написать с максимально экономичным отношением к ресурсам и минимальной зависимостью от системы, но большим опытом в php не обладаю, поэтому — буду признателен рекомендациям по оптимизации.

Скрипт отвечает как на POST, так и GET запросы. Разницы между ними нет. **Так же отвечает на параметры, переданные в командной строке**. Например: `php ./rpc.php get_list` или `php ./rpc.php add_task http://goo.gl/5Qi0Xs`.


Особенности и настройки клиентской части
----
Не используются «новые html5 теги», но используются свойства css3 для оформления прогресc-бара загрузок и адаптива. Дизайн выполнен в минималистичном стиле. При отсутствии задач в центре страницы располагается поле для добавления адреса закачки, если задачи имеются — это поле смещается вверх страницы, и ниже располагаются задачи.

Все запросы — асинхронные (без перезагрузки страницы). Дизайн страницы — адаптивный:
![screenshot](http://habrastorage.org/files/479/aec/f73/479aecf737f647e485fb325671fe6df5.png)

Изменение состояния отображается также в заголовке вкладки (окна):
![screenshot](http://habrastorage.org/files/e8c/78d/52f/e8c78d52ff494d0b9f6d7aa56bf957b8.png)

В нижней части страницы располагается javascript-закладка («Download this»), переместив которую в панель закладок браузера можно одним кликом добавлять новые задачи (при клике будет добавлена активная вкладка; если открыта вкладка с видеофайлом и будет нажата эта «закладка» на панели закладок — будет добавлена задача на скачивание этого видеофайла):
![screenshot](http://habrastorage.org/files/462/4d0/f9c/4624d0f9c3494fee9987e4ba757a423b.png)

Есть функция одновременного добавления нескольких задач, для этого в поле вписывай несколько строк (одна задача - одна строка).

Для того, что бы задать имя сохраняемого файла добавь к URL в GUI строку вида "` -> filename.ext`" ("пробел, тире, больше, пробел" полный вид запроса при этом будет "`htttp://somehost.io/oldfilename.zip -> newfilename.zip`"). Если указанное указанный файл имеется - он будет перезаписан.

Весь javascript код документа расположен в файле core.js. В верхней его части располагаются основные настройки:

 - `addTasksLimitCount = 5` - лимит одновременно добавляемых задач;
 - `updateStatusInterval = 5 * 1000` - интервал обновления данных на  открытой вкладке (будьте аккуратны с этим параметром на слабых серверах);
 - `DebugMode = false` - режим отладки, выводится отладочная информация в console.log;
 - `CheckForUpdates = true` - функция автоматической проверки обновления;
 - `prc = 'rpc.php'` - путь до скрипта серверной части.

Описывать функциональные моменты смысла особого не вижу, но скажу — функции разделены на логические группы, скрипт не минифицирован, комментарии имеют место быть.

При нажатии на F5 происходит принудительное обновление задач, страница перезагружается только по нажатию на кнопку обновления в браузере.

### [Установка / обновление]

### [История изменений]

#### [Расширения для браузеров]:

 * [Wget GUI Light - Google chrome Extension]
 * Firefox (в планах)
 * Opera (в планах)


#### [Пост на хабре]

В случае возникновения ошибок — [задайте вопрос].

### Лицензия: **MIT**
Copyright © 2014 Samoylov Nikolay

Данная лицензия разрешает лицам, получившим копию данного программного обеспечения и сопутствующей документации (в дальнейшем именуемыми «Программное Обеспечение»), безвозмездно использовать Программное Обеспечение без ограничений, включая неограниченное право на использование, копирование, изменение, добавление, публикацию, распространение, сублицензирование и/или продажу копий Программного Обеспечения, также как и лицам, которым предоставляется данное Программное Обеспечение, при соблюдении следующих условий:

Указанное выше уведомление об авторском праве и данные условия должны быть включены во все копии или значимые части данного Программного Обеспечения.

ДАННОЕ ПРОГРАММНОЕ ОБЕСПЕЧЕНИЕ ПРЕДОСТАВЛЯЕТСЯ «КАК ЕСТЬ», БЕЗ КАКИХ-ЛИБО ГАРАНТИЙ, ЯВНО ВЫРАЖЕННЫХ ИЛИ ПОДРАЗУМЕВАЕМЫХ, ВКЛЮЧАЯ, НО НЕ ОГРАНИЧИВАЯСЬ ГАРАНТИЯМИ ТОВАРНОЙ ПРИГОДНОСТИ, СООТВЕТСТВИЯ ПО ЕГО КОНКРЕТНОМУ НАЗНАЧЕНИЮ И ОТСУТСТВИЯ НАРУШЕНИЙ ПРАВ. НИ В КАКОМ СЛУЧАЕ АВТОРЫ ИЛИ ПРАВООБЛАДАТЕЛИ НЕ НЕСУТ ОТВЕТСТВЕННОСТИ ПО ИСКАМ О ВОЗМЕЩЕНИИ УЩЕРБА, УБЫТКОВ ИЛИ ДРУГИХ ТРЕБОВАНИЙ ПО ДЕЙСТВУЮЩИМ КОНТРАКТАМ, ДЕЛИКТАМ ИЛИ ИНОМУ, ВОЗНИКШИМ ИЗ, ИМЕЮЩИМ ПРИЧИНОЙ ИЛИ СВЯЗАННЫМ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ ИЛИ ИСПОЛЬЗОВАНИЕМ ПРОГРАММНОГО ОБЕСПЕЧЕНИЯ ИЛИ ИНЫМИ ДЕЙСТВИЯМИ С ПРОГРАММНЫМ ОБЕСПЕЧЕНИЕМ.

### 3rd party:

 * Библиотека [jquery]
 * Парсер ссылок [url.js]
 * Плагин уведомлений [notifIt]
 * [CSS3 progress bar]
 * Плагин для работы с плюшками [jquery.cookie.js]
 * Попап-плагин [bpopup]

[Расширения для браузеров]:https://github.com/tarampampam/wget-gui-light/tree/master/browser-extension
[Wget GUI Light - Google chrome Extension]:https://chrome.google.com/webstore/detail/wget-gui-light/dbcjcjjjijkgihaddcmppppjohbpcail
[Пост на хабре]:http://habrahabr.ru/post/234353/
[wget]:https://ru.wikipedia.org/wiki/Wget
[Установка / обновление]:https://github.com/tarampampam/wget-gui-light/blob/master/HowUpdate.md
[История изменений]:https://github.com/tarampampam/wget-gui-light/blob/master/cahngeslog.md
[задайте вопрос]:https://github.com/tarampampam/wget-gui-light/issues/new
[notifIt]:https://dl.dropboxusercontent.com/u/19156616/ficheros/notifIt!-1.1/index.html
[jquery]:http://jquery.com/
[url.js]:http://habrahabr.ru/post/232073/
[CSS3 progress bar]:http://css-tricks.com/css3-progress-bars/
[jquery.cookie.js]:https://github.com/carhartl/jquery-cookie
[bpopup]:http://dinbror.dk/bpopup/
