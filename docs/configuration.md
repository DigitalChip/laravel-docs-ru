# Laravel 8 · Конфигурирование

- [Введение](#introduction)
- [Конфигурация окружения](#environment-configuration)
    - [Типы переменных окружения](#environment-variable-types)
    - [Получение конфигурации окружения](#retrieving-environment-configuration)
    - [Определение текущего окружения](#determining-the-current-environment)
- [Доступ к значениям конфигурации](#accessing-configuration-values)
- [Кеширование конфигурации](#configuration-caching)
- [Режим отладки](#debug-mode)
- [Режим обслуживания](#maintenance-mode)

<a name="introduction"></a>
## Введение

Все конфигурационные файлы фреймворка Laravel хранятся в каталоге `config`. Каждый параметр задокументирован, поэтому не стесняйтесь просматривать эти файлы и знакомиться с доступными вам вариантами.

Конфигурационные файлы позволяют настраивать такие вещи, как информация о подключении к базе данных, информация о почтовом сервере, а также другие основные параметры, например, часовой пояс приложения и ключ шифрования.

<a name="environment-configuration"></a>
## Конфигурация окружения

Часто бывает полезно иметь различные конфигурации в зависимости от окружения, в котором выполняется приложение. Например, по желанию можно использовать разные драйверы кеша в локальном и эксплуатационном окружении.

Чтобы упростить это, Laravel использует библиотеку [DotEnv](https://github.com/vlucas/phpdotenv) PHP. В корневом каталоге вашего нового приложения будет содержаться файл `.env.example`, определяющий множество основных переменных окружения. Этот файл будет автоматически скопирован в `.env` в процессе установки Laravel.

Файл `.env` Laravel по умолчанию содержит некоторые основные значения конфигурации, которые могут зависеть от того, работает ли ваше приложение локально или на конечном веб-сервере. Эти значения могут быть получены из различных конфигурационных файлов каталога `config` Laravel с помощью глобальной функции `env()` Laravel.

Если вы работаете в команде, то вы можете не исключать файл `.env.example` из своего приложения. Размещая значения-заполнители в этот файл, другие разработчики в вашей команде могут четко видеть, какие переменные окружения необходимы для запуска вашего приложения.

> {tip} Любая переменная в вашем файле `.env` может быть переопределена внешними переменными окружения, такими как переменные окружения уровня сервера или системы.

<a name="environment-file-security"></a>
#### Безопасность файлов окружения

Ваш файл `.env` не должен быть привязан к системе контроля версий вашего приложения, поскольку каждому разработчику / серверу, использующему ваше приложение, может потребоваться другая конфигурация окружения. Кроме того, это будет угрозой безопасности в случае, если злоумышленник получит доступ к вашему репозиторию системы управления версиями, поскольку любые конфиденциальные учетные данные будут раскрыты.

<a name="environment-variable-types"></a>
### Типы переменных окружения

Все переменные в файлах `.env` обычно анализируются как строки, поэтому были созданы некоторые зарезервированные значения, позволяющие вам возвращать более широкий диапазон типов из функции `env()`:

Значение `.env`  | Значение `env()`
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

Если вам нужно определить переменную окружения со значением, содержащим пробелы, то вы можете сделать это, заключив значение в двойные кавычки:

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### Получение конфигурации окружения

Все переменные, перечисленные в этом файле, будут загружены в суперглобальную переменную `$_ENV` PHP, когда ваше приложение получит запрос. Однако вы можете использовать помощник `env()` для получения значений из переменных ваших конфигурационных файлов. Фактически, если вы просмотрите файлы конфигурации Laravel, вы заметите, что многие параметры уже используют этот помощник:

    'debug' => env('APP_DEBUG', false),

Второе значение, переданное в функцию `env`, является «значением по умолчанию». Это значение будет возвращено, если для данного ключа не существует переменной окружения.

<a name="determining-the-current-environment"></a>
### Определение текущего окружения

Текущее окружение приложения определяется с помощью переменной `APP_ENV` из вашего файла `.env`. Вы можете получить доступ к этому значению через метод `environment` [фасада](facades.md) `App`:

    use Illuminate\Support\Facades\App;

    $environment = App::environment();

Вы также можете передать аргументы методу `environment`, чтобы определить, соответствует ли окружение переданному значению. Метод вернет `true`, если окружение соответствует любому из указанных значений:

    if (App::environment('local')) {
        // Локальное окружение ...
    }

    if (App::environment(['local', 'staging'])) {
        // Окружение либо локальное, либо промежуточное ...
    }

> {tip} Определение текущего окружения приложения может быть отменено путем определения переменной окружения `APP_ENV` на уровне сервера.

<a name="accessing-configuration-values"></a>
## Доступ к значениям конфигурации

Вы можете легко получить доступ к своим значениям конфигурации, используя глобальный помощник `config` из любого места вашего приложения. Доступ к значениям конфигурации можно получить с помощью «точечной нотации», который включает имя файла и параметр, к которому вы хотите получить доступ. Также может быть указано значение по умолчанию, которое будет возвращено, если параметр конфигурации отсутствует:

    $value = config('app.timezone');

    // Получить значение по умолчанию, если значение конфигурации не существует ...
    $value = config('app.timezone', 'Asia/Seoul');

Чтобы установить значения конфигурации во время выполнения скрипта, передайте массив помощнику `config`:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Кеширование конфигурации

Чтобы ускорить работу вашего приложения, вы должны кешировать все конфигурационные файлы в один файл с помощью команды `config:cache` Artisan. Это объединит все конфигурационные параметры вашего приложения в один файл, который может быть быстро загружен фреймворком.

Обычно вы должны запускать команду `php artisan config:cache` как часть процесса развертывания эксплуатационного режима. Команду не следует запускать во время локальной разработки, поскольку  конфигурационные параметры часто нужно будет изменять в ходе разработки вашего приложения.

> {note} Если вы выполняете команду `config:cache` в процессе развертывания, то вы должны быть уверены, что вызываете функцию `env()` только из ваших файлов конфигурации. После кэширования конфигурации файл `.env` не будет подгружаться; следовательно, функция `env()` будет возвращать только внешние переменные окружения системного уровня.

<a name="debug-mode"></a>
## Режим отладки

Параметр `debug` в конфигурационном файле `config/app.php` определяет, сколько информации об ошибках фактически отображается конечному пользователю. По умолчанию этот параметр установлен с учетом значения переменной `APP_DEBUG` окружения, расположенной в вашем файле `.env`.

Для локальной разработки вы должны установить для переменной `APP_DEBUG` окружения значение `true`. **В эксплуатационном режиме это значение всегда должно быть `false`. Если для этой переменной будет установлено значение `true`, то вы рискуете раскрыть конфиденциальные значения конфигурации конечным пользователям вашего приложения.**

<a name="maintenance-mode"></a>
## Режим обслуживания

Когда ваше приложение находится в режиме обслуживания, то для всех запросов к приложению будет отображаться специальная страница. Это позволяет легко «отключить» ваше приложение во время его обновления или технического обслуживания. Проверка режима обслуживания включена в стек посредников по умолчанию для вашего приложения. Если приложение находится в режиме обслуживания, то будет выброшено исключение `Symfony\Component\HttpKernel\Exception\HttpException` с `503` кодом состояния.

Чтобы включить режим обслуживания, выполните команду `down` Artisan:

    php artisan down

Если вы хотите, чтобы HTTP-заголовок `Refresh` отправлялся со всеми ответами режима обслуживания, то вы можете указать параметр `refresh` при вызове команды `down`. Заголовок `Refresh` проинструктирует браузер автоматически обновлять страницу через указанное количество секунд:

    php artisan down --refresh=15

Вы также можете передать команде `down` параметр `retry`, значение которого будет установлено в заголовке `Retry-After` HTTP, хотя браузеры обычно игнорируют этот заголовок:

    php artisan down --retry=60

<a name="bypassing-maintenance-mode"></a>
#### Обход режима обслуживания

Находясь в режиме обслуживания, вы можете использовать параметр `secret`, чтобы указать токен для обхода режима обслуживания:

    php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"

После перевода приложения в режим обслуживания, вы можете перейти по URL-адресу приложения, с учетом этого токена, и Laravel выдаст вашему браузеру файл куки для обхода режима обслуживания:

    https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515

При доступе к этому скрытому маршруту вы будете перенаправлены на маршрут `/` приложения. Как только куки будет отправлен вашему браузеру, вы сможете просматривать приложение в обычном режиме, как если бы оно не находилось в режиме обслуживания.

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Предварительный рендеринг шаблона режима обслуживания

Если вы используете команду `php artisan down` во время развертывания, то ваши пользователи могут иногда сталкиваться с ошибками, если они обращаются к приложению во время обновления ваших зависимостей Composer или других компонентов фреймворка. Это происходит потому, для определения режима обслуживания и отображения шаблона режима обслуживания с помощью движка шаблонов должна быть загружена значительная часть фреймворка Laravel.

По этой причине Laravel позволяет в самом начале цикла запроса отобразить шаблон режима обслуживания. Этот шаблон отображается перед загрузкой любых зависимостей вашего приложения. Вы можете выполнить предварительный рендеринг шаблона по вашему выбору, используя параметр `render` команды `down`:

    php artisan down --render="errors::503"

<a name="redirecting-maintenance-mode-requests"></a>
#### Перенаправление запросов режима обслуживания

В режиме обслуживания Laravel будет отображать шаблон режима обслуживания для всех URL-адресов приложения, к которым пользователь попытается получить доступ. Если хотите, то вы можете указать Laravel перенаправлять все запросы на определенный URL. Это может быть выполнено с помощью параметра `redirect`. Например, вы можете перенаправить все запросы на URI `/`:

    php artisan down --redirect=/

<a name="disabling-maintenance-mode"></a>
#### Отключение режима обслуживания

Чтобы отключить режим обслуживания, используйте команду `up`:

    php artisan up

> {tip} Вы можете определить свой шаблон режима обслуживания в `resources/views/errors/503.blade.php`.

<a name="maintenance-mode-queues"></a>
#### Режим обслуживания и очереди

Пока ваше приложение находится в режиме обслуживания, [поставленные в очередь задания](queues.md) обрабатываться не будут. Задания продолжат обрабатываться в обычном режиме после выхода приложения из режима обслуживания.

<a name="alternatives-to-maintenance-mode"></a>
#### Альтернативы режиму обслуживания

Поскольку режим обслуживания требует, чтобы ваше приложение простаивало несколько секунд, то рассмотрите альтернативы, например, [Laravel Vapor](https://vapor.laravel.com) и [Envoyer](https://envoyer.io) для выполнения развертывания с нулевым временем простоя.
