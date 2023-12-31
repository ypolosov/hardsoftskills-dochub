# Домашнее задание 1 ([source](https://docs.google.com/document/d/1GH83Q1nxd1frC1rNn-7i03mQpbfwV6OCW_XqCBKaybY/edit?usp=sharing))

## Условие:

- 1 млн новых пользователей в год, каждый пользователь в год в среднем сокращает 100 ссылок. 
- Сокращения нужно хранить 3 года. 
- Перенаправление на целевой адрес должно происходить не более чем за 0.2с. 
- Статистику по переходам отслеживать нужно. 
- Релизы системы должны происходить бесшовно.
- Посещаемость ссылок - в среднем 10 обращений в день к одной ссылке.
- *Отказ любых двух узлов должен ухудшать производительность системы не более чем на 0.1с.

## Задание:

- Сделать необходимые вычисления на салфетке.
- Выбрать и обосновать схему хранения данных, удаления устаревших ссылок, хранения/обновления статистики и других необходимых функциональных компонентов. 
- *Выбрать и обосновать серверную инфраструктуру, все необходимые компоненты, технологии, библиотеки. 
- Обосновать надежность системы.
- Выбрать и обосновать схему релизов, и инструменты релизов.
- Сформулировать и обосновать требования к команде разработки.
- Выбрать и обосновать окружения и инструменты, необходимые для разработки.
- Выбрать и обосновать хостинг провайдера, оценить расходы на сервера для всех окружений, включая production.


## Этапы решения

### 1. Вопросы по постановке задачи
 - вопросы по бизнес-домену.
 - вопросы по околотехническим аспектам бизнеса (availability, количество обращений, данных...).
 - перед этим разделом нужно тщательно изучить постановку и предметную область

#### Решение

##### Вопросы по бизнес-домену (функциональные требования)
- Кто наши пользователи? (люди, другие сервисы)
- Какие роли пользователей предполагаются в системе? (пользователь, администратор, аналитик) 
- Какие примеры типичных юскейсов использования сервиса?
- Где регионально сервис будет представлять услуги?
- Нужна ли AuthN / AuthZ ?
- Какие действия могут осуществлять разные роли? (CRUD, просмотр статистики, возможность делать аналитические запросы)
- Какое кол-во пользователей предполагается будет обрабатывать система в год/месяц/день?
- Предполагаются ли сезонные всплески? (праздники, распродажи)
- Какие символы может содержать ссылка?
- Какое кол-во ссылок создается каждый день?
- Какое кол-во ссылок читается каждый день?
- Как долго нужно хранить ссылки?

##### Вопросы по околотехническим аспектам бизнеса (нефункциональные требования)
- Сколько активных пользователей в год/месяц/день?
- Как быстро должно происходить перенаправление по ссылке?
- Нужно ли отслеживать статистику по переходам? (кол-во созданных ссылок, кол-во уникальных пользователей)
- Какая стратегия деплоя сервиса? (recreate, rolling update, blue/green, canary)
- Нужно ли поддерживать несколько версий сервиса? сколько? как долго?
- Какие типы ссылок будут сокращаться? Будут ли это ссылки на сторонние сайты или только на внутренние страницы?
- Какой лимит на хранение ссылки?
- Какие данные необходимо хранить для каждой ссылки? (дата создания, дата истечения срока действия)
- Какие метрики необходимо отслеживать для каждой ссылки? (количество уникальных пользователей, время нахождения на странице)
- Какие метрики необходимо отслеживать для всей системы? (количество созданных ссылок, количество переходов, количество уникальных пользователей)
- Нужно ли генерить каждый раз новую короткую ссылку на одинаковую длинную ссылку?
- Какая стратегия защиты сервиса от перегрузок? (rate limiter)
- Должен ли сервис проверять ссылки на наличие вирусов и других угроз?
- Какие стек и инструменты уже используются в компании? Должны ли мы отталкиваться от существующего или мы свободны в выборе? (on-premises, cloud vendors)
- Какая стратегия восстановления сервиса при отказе?
- Что по CAP? CA? AP? CP?
- SRE. Какие SLI будем отслеживать? Какой наш внутренний SLO по сервису? Какой SLA мы готовы предоставить потребителям?
- Какие еще нефункциональные требования важны? (availability, reliability, testability, scalability, security, agility, falut tolerance, elasticity, recoverability, performance, deployability, learnability)
- Есть ли лимит на какието ресурсы? Деньги, люди, время?

### 2. Выводы и предположения
- краткое описание для чего именно бизнесу эта система, какие его проблемы решает, какие цели выполняет
- отвечают на вопросы наиболее адекватным для вашего понимания домена способом
- если есть сильные отличия в архитектуре при разных ответах, стоит сделать несколько вариантов ответов на вопрос и под них несколько вариантов архитектуры

#### Решение
Учитывая большое кол-во неотвеченных вопросов, а следовательно ограниченное представление о системе, решено на данном этапе спроектировать минимальное решение покрывающее заявленные на данный момент требования.

##### Бизнес архитектура
- Миссия
    - Сделать жизнь пользователей проще, предоставив им возможность использовать короткие и удобные ссылки
- Цели
    - Увеличить трафик
    - Увеличить конверсию
- Функции
    - Создание коротких ссылок
    - Перенаправление на исходный адрес при клике по короткой ссылке
    - Отслеживание статистики по переходам
    - Хранение ссылок на протяжении 3 лет
- Процессы
    - Создание короткой ссылки
        - Получение длинной ссылки: Пользователь вводит длинную ссылку, которую нужно сократить
        - Генерация короткой ссылки: Система генерирует короткую ссылку, которая будет использоваться вместо длинной ссылки
        - Хранение ссылки: Система сохраняет короткую ссылку в базе данных, чтобы ее можно было использовать в дальнейшем
        - Отображение короткой ссылки: Система отображает короткую ссылку пользователю, который может использовать ее вместо длинной ссылки
    - Переход по короткой ссылке
        - Перехват клика: Система перехватывает клик пользователя по короткой ссылке
        - Поиск ссылки: Система ищет соответствующую длинную ссылку в базе данных
        - Перенаправление: Система перенаправляет пользователя на исходный адрес, связанный с длинной ссылкой
    - Отслеживание статистики переходов по ссылке 
        - Статистика: Система сохраняет информацию о переходе пользователя по короткой ссылке в базе данных
    - Хранение ссылок на протяжении 3 лет
        - Сбор ссылок: Система собирает все короткие ссылки, которые были созданы за последние 3 года
        - Хранение ссылок: Система сохраняет все собранные ссылки в базе данных
        - Архивирование ссылок: Система архивирует все ссылки, которые были созданы более 3 лет назад. Архивирование позволяет сохранить ссылки для аналитических целей


### 3. Оценки "на салфетке"
 - исходя из бизнес-домена и предположений, оценить с погрешностью не более 3-5x необходимые для построения архитектуры параметры
 - чаще всего это объем данных и количество запросов, нагрузка на различные компоненты системы
 - определяем какие параметры исходя из того, что важно бизнесу, без чего технически система не сможет приносить пользу
 - ballpark estimates, back of the envelope calculation

#### Решение

##### Back of the envelope calculation
- Входные данные
    - Начальное кол-во пользователей (Starting Users) = 0
    - Период на который рассчитывается нагрузка на сервис (Number of Years) = 5 лет
    - Общий прирост пользователей в год (Average Annual Growth Rate) = 1M
    - Кол-во сокращений ссылок одним пользователем в год = 100
    - Кол-во обращений к одной ссылке в день = 10
    - Средняя длинна ссылки = 100 символов (взято из интернета)
    - Средний размер REST POST запроса = 100B + 100B = 200B 
    - Средний размер REST GET запроса = 100B
- Расчетные данные
    - YAU (Yearly Active Users) = 0 + (1M * 0 * 5) = 5M
        - Active Users = Starting Users + (Average Annual Growth Rate * Starting Users * Number of Years)
    - MAU (Monthly Active Users) = 5M / 12 ~ 420T
    - DAU (Daily Active Users) = 420T / 30 ~ 15T
    - Кол-во сокращений ссылок за 5 лет = 1M * 100 + 2M * 100 + 3M * 100 + 4M * 100 + 5M * 100 = 100M + 200M + 300M + 400M + 500M = 1500M
    - Write RPS (Requests Per Second) = 500M / 12 / 30 / 24 / 60 / 60 ~ 20
    - Read RPS (Requests Per Second) = 1500M * 10 / 24 / 60 / 60 ~ 200T
    - Размер записи для в DB хранения ссылки = 100 * 1B = 100B
    - Размер DB для хранения всех ссылок = 100B * 1500M = 150Gb
    - Trafic Per Second = 200T * 100B + 20 * 200B ~ 20Mb * 8 ~ 200Mbit/s

### 4. Выбор подхода к архитектуре
 - какие принципы и критерии и почему важны именно при построении этой системы
 - например, для финтеха - консистентность данных, для счетчика посещений - скорость ответа, для корпоративного приложения - безопасность, для стартапа - гибкость						
 - также здесь нужно указать на наиболее сложные и проблемные места, которые являются причиной серьезных рисков в жц системы, и на послабления требований, который можно использовать
 - перед этим разделом стоит изучить архитектуру аналогичных проектов, доступную открыто				

#### Решение
- CAP. Согласно теории CAP в распределенных системах можно одновременно только два из трех свойств. Для сервиса "Сокращатель ссылок" необходимо прежде всего поддерживать доступность и разделение, следственно это система категории - AP. То есть система должна быть доступна для клиентов в любое время даже если произошел сбой в системе ( A) и она должна продолжать работать даже если произошел сбой в сети ( P). Однако при этом могут возникнуть проблемы с согласованностью данных ( C) которая может быть решена компромиссным путем - eventually consistency. Что значит мы заведомо предполагаем, что данные могут быть несогласованными на короткое время, но в конечном итоге будут согласованы
- Cloud vs On-Premises. Использование облачных провайдеров освобождает нас от ряда забот по поднятию инфраструктуры с нуля. Поэтому является приоритетным как минимум на начальном этапе развития сервиса, что позволит сфокусироваться на реализации бизнес-задач и сократит time-to-market.
- Managed vs Self-hosted. Учитывая сто мы начинаем строить сервис from scratch и мы свободны в выборе, то выбор облачных (managed) решений освобождает нас от множества проблем связанных с поддержкой нефункциональных требований, хотя и может оказаться дороже в долгосрочной перспективе. Однако реализация таких сервисов часто скрыта от пользователей, что лишает нас интереса с точки зрения обучения. Поэтому принято решение минимизировать использование managed сервисов в дизайне.
- Оркестрация. Нужен механизм оркестрации который поможет обстрагиваться от задач оркестрации зоопарка серверов на уровне hardware и зоопарка сервисов на уровне software, но в тоже время позволит сохранить контроль над их конфигурацией, а также позволит в большей степени сфокусироваться на задачах дизайна системы. Kubernetes как стандарт де-факто в индустрии позволит соблюсти баланс в долгосрочной перспективе.
- Load balancing. Нужен механизм позволяющий распределять нагрузку входящих запросов между меняющимся ко-вом воркеров, при этом абстрагироваться от деталей, но сохранить контроль над возможностью настройки.
- Caching. Нужен механизм позволяющий переиспользовать результаты предыдущих вычислений не прибегая к их повтору при соблюдении ряда предусловии.
- Sharding. Чтобы облегчить параллельную работу с данными и повысить производительность, необходим механизм позволяющий распределять данные между множественными DB серверами
- Replication. Чтобы свести к минимуму эффект единой точки отказа в части работы с данными, нужен механизм синхронизации копий данных на другие DB сервера
- Message queues. Уменьшить связанность и привнести асинхронную природу в межсервисном взаимодействии поможет event-driven подход
- CQRS. Разделение флоу чтения и записи данных, расширяет возможности по их независимому дизайну и масштабированию.

### 5. Архитектурные диаграммы и пояснения
- диаграмма придуманной вами архитектуры - основные компоненты и потоки данных
- диаграмма должна показывать за счет чего эта система будет выполнять указанные потребности бизнеса
- диаграмм может быть несколько, для разных компонентов/подсистем, разные view
- больше всего интересуют context, container и component по С4, code по C4 нас будет интересовать очень редко - только если там хитрые алгоритмы, которые влияют на архитектуру (пример хитрых алгоритмов - алгоритмы консенсуса, или ml модели и их обучение)
- https://c4model.com/

#### Решение

([C4model](https://miro.com/app/board/uXjVNDuNjo4=/?share_link_id=288427604355))

### 6. Обоснование конкретных инструментов
- почему выбрали именно эти базы и др компоненты, и почему именно эти подходы лучше всего в этой ситуации
- наилучший способ обосновать - сделать таблицу сравнений нескольких вариантов

#### Решение
... здесь будет решение ...


### 7. Отдельные аспекты
- здесь то что важно, но что не попало в предыдущие разделы
- например, оценка стоимости инфраструктуры, или отдельные алгоритмы/компоненты балансировки, или краткие описания малоизвестных инструментов...

#### Решение
- Организационная структура
    - Менеджмент
        - CEO
        - CTO
    - Команда разработки
        - Разработчики
        - Тестировщики
        - Менеджеры проектов
    - Команда маркетинга
        - Маркетологи
        - Копирайтеры
        - Дизайнеры
    - Команда поддержки
        - DevOps
        - SRE
        - Специалисты поддержки клиентов
- Цена за трафик и железо
