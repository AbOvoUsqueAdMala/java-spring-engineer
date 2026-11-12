# Источники, сравнение и принятые решения

Дата исследования: **22 октября 2026 года**. Это обоснованная выборка популярных и профильных проектов, а не доказанный мировой рейтинг скиллов. Звёзды относятся к репозиторию целиком, не к конкретному скиллу и не к качеству его советов. Содержание проверялось по исходным `SKILL.md`, отдельным файлам кода, README и официальной документации; каталоги скиллов использовались только для поиска.

## Как отбирались источники

Сравнивались применимость к Java/Spring, конкретность инструкций, уважение к существующему проекту, работа с отказами, проверяемость и актуальность интеграций. Популярность помогла найти распространённые решения; профильные небольшие проекты добавлены за конкретную специализацию. По метаданным GitHub на дату проверки все пять выбранных репозиториев скиллов не архивированы. Дата последнего push отражает активность репозитория, а не гарантирует обновление каждого скилла.

| Репозиторий скиллов | Звёзды, снимок GitHub API | Последний push, UTC | Роль в сравнении |
|---|---:|---|---|
| [affaan-m/ECC](https://github.com/affaan-m/ECC) | 285 972 | 2026-10-20 | Популярная универсальная коллекция; изучены Spring patterns и security |
| [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) | 11 997 | 2026-10-21 | Популярная коллекция; изучены Java Architect и Spring Boot Engineer |
| [jdubois/dr-jskill](https://github.com/jdubois/dr-jskill) | 344 | 2026-10-13 | Специализированный генератор Java/Spring-приложений |
| [SivaLabs agent skills](https://github.com/sivaprasadreddy/sivalabs-agent-skills) | 189 | 2026-10-12 | Модульность Spring-приложений и стратегия тестирования |
| [Amplicode/spring-skills](https://github.com/Amplicode/spring-skills) | 132 | 2026-09-06 | Работа с моделью и соглашениями конкретного проекта |

## Семь изученных скиллов: что сохранено и что изменено

Оценки в двух последних колонках — инженерные выводы этого исследования, а не утверждения авторов о собственных недостатках. Они относятся к прочитанным версиям файлов.

| Исходный скилл | Полезное | Изменение в новом скилле |
|---|---|---|
| [Jeffallan: java-architect](https://github.com/Jeffallan/claude-skills/blob/main/skills/java-architect/SKILL.md) | Разделение тем, миграции, внимание к транзакциям и проверке | Убраны универсальные требования Java 21, Boot 3.x, DDD и покрытия 85%; версия и архитектура определяются проектом |
| [Jeffallan: spring-boot-engineer](https://github.com/Jeffallan/claude-skills/blob/main/skills/spring-boot-engineer/SKILL.md) | Constructor injection, конфигурация, web/data/security/test references | Пример отдаёт сущность Product через REST и не ограничивает список; введены явные границы API и ограничения запросов. Работа не заканчивается обязательным развёртыванием |
| [ECC: springboot-patterns](https://github.com/affaan-m/ECC/blob/main/skills/springboot-patterns/SKILL.md) | DTO, транзакции, наблюдаемость, организация backend | Пример retry ловит все Exception и ждёт без jitter; заменён критериями transient failure, идемпотентности, общего deadline и одного владельца retry. Учтена ёмкость локальных карт/кэшей |
| [ECC: springboot-security](https://github.com/affaan-m/ECC/blob/main/skills/springboot-security/SKILL.md) | Авторизация, validation, secrets, CSRF и CORS | Для стандартного OAuth2 выбран встроенный Resource Server вместо шаблонного custom JWT filter; добавлены audience, права на конкретный объект/tenant и различие bearer header/bearer cookie |
| [SivaLabs: spring-boot](https://github.com/sivaprasadreddy/sivalabs-agent-skills/blob/main/skills/spring-boot/SKILL.md) | Модули, разные уровни тестирования, реальная БД, ArchUnit | Сохранены тесты по границам поведения; Testcontainers нужен для чувствительных к БД свойств, а новые плагины, Taskfile и конкретный набор уровней тестов не навязываются каждой задаче |
| [Dr JSkill](https://github.com/jdubois/dr-jskill/blob/main/SKILL.md) | Учёт совместимости, генерация из Spring Initializr, отсутствие обязательного пустого service layer | Не перенесены обязательные Java 25, Maven, PostgreSQL, frontend, `.properties` и native-image support. Они допустимы по требованиям, но не являются универсальными практиками |
| [Amplicode: spring-data-jpa](https://github.com/Amplicode/spring-skills/blob/main/skills/spring-data-jpa/SKILL.md) | Изучение соглашений сущностей и транзакций до изменений | Новый скилл самодостаточен: отсутствие Amplicode MCP не останавливает работу; контекст можно получить из исходников. Установка IDE-плагина не является предпосылкой |

## Сопоставление с Java/Spring-репозиториями

Изучались выбранные файлы и документация, а не весь исходный код каждого проекта. Число звёзд ниже — тот же снимок GitHub API. Petclinic — учебный образец; библиотечный код Guava также нельзя автоматически переносить в архитектуру приложения.

| Проект | Звёзды | Проверенный материал | Вывод для скилла |
|---|---:|---|---|
| [Spring Boot](https://github.com/spring-projects/spring-boot) | 81 754 | [README](https://github.com/spring-projects/spring-boot/blob/main/README.adoc), требования к системе, Testcontainers, Actuator | Использовать платформенные defaults/BOM, но уступать конкретным требованиям; различать JDK сборки самого Boot и минимум для приложения |
| [Google Guava](https://github.com/google/guava) | 51 916 | [README](https://github.com/google/guava/blob/master/README.md), [ImmutableList.java](https://github.com/google/guava/blob/master/guava/src/com/google/common/collect/ImmutableList.java) | Явные контракты владения, nullability и совместимости API. Не добавлять Guava, если хватает JDK |
| [Resilience4j](https://github.com/resilience4j/resilience4j) | 10 791 | [README](https://github.com/resilience4j/resilience4j/blob/master/README.adoc) | Разделять состояние защит по зависимостям; выбирать только нужные механизмы; проверять совместимость версии с JDK |
| [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) | 9 595 | [OwnerController.java](https://github.com/spring-projects/spring-petclinic/blob/main/src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java) | Feature package, constructor injection, validation, pagination. В образце контроллер обращается к репозиторию: пустой service layer не универсальная обязанность. Его HTML form binding не шаблон публичного REST API |
| [Netflix concurrency-limits](https://github.com/Netflix/concurrency-limits) | 3 620 | [README](https://github.com/Netflix/concurrency-limits/blob/main/README.md), [VegasLimitTest.java](https://github.com/Netflix/concurrency-limits/blob/main/concurrency-limits-core/src/test/java/com/netflix/concurrency/limits/limit/VegasLimitTest.java) | Контролировать in-flight work и перегрузку. Тесты проверяют увеличение/уменьшение лимита на заданных измерениях — полезен принцип детерминированной проверки, не копирование JUnit-версии или порогов |
| [Netflix DGS](https://github.com/Netflix/dgs-framework) | 3 401 | [README и compatibility table](https://github.com/Netflix/dgs-framework/blob/master/README.md) | Прямой пример Spring Boot-проекта Netflix: schema/query tests и совместимость интеграций. GraphQL применяется при соответствующей задаче |
| [Spring Modulith](https://github.com/spring-projects/spring-modulith) | 1 202 | [readme.adoc](https://github.com/spring-projects/spring-modulith/blob/main/readme.adoc), verification и events | Проверяемые модульные границы и документированная надёжная публикация событий как альтернатива преждевременному разделению на микросервисы |
| [Netflix Hystrix](https://github.com/Netflix/Hystrix) | 24 480 | [Раздел о статусе](https://github.com/Netflix/Hystrix/blob/master/README.md) | Контрпример отбора только по звёздам: maintenance mode. Сохранены идеи изоляции отказов, но библиотека не предлагается как выбор для нового сервиса |

`archived=false` и свежий push сами по себе не доказывают активную поддержку продукта: в случае Hystrix решающим является прямо опубликованный статус.

## Практики компаний и первичная документация

- **Netflix:** ограничения конкуренции, отказоустойчивость и тестирование контрактов DGS взяты из публичных проектов выше. Универсальная архитектура всех внутренних сервисов Netflix не предполагается.
- **Amazon:** [Timeouts, retries, and backoff with jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) обосновывает ограниченные повторы, jitter, таймауты и осторожность с побочными эффектами. В скилле эти принципы применены к Java HTTP-клиентам и обработчикам сообщений.
- **Google:** Guava даёт примеры явных контрактов и дисциплины совместимости. Для Java-кода применяется публичный [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) с пользовательским исключением: отступы выполняются символами табуляции. Это адаптированный стиль, а не строгое соответствие Google Java Style, не утверждение о внутренних стандартах всех команд Google и не требование подключать Guava.
- **Zalando:** [RESTful API and Event Guidelines](https://opensource.zalando.com/restful-api-guidelines/) использованы для контрактов и эволюции API/событий; корпоративные hostname/naming conventions не перенесены.
- **Spring:** проверены [системные требования](https://docs.spring.io/spring-boot/system-requirements.html), [транзакционная семантика](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html), [JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html), [CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html), [ProblemDetail](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html), [Testcontainers](https://docs.spring.io/spring-boot/reference/testing/testcontainers.html), [Actuator/probes](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html), [Modulith verification](https://docs.spring.io/spring-modulith/reference/verification.html) и [event publication](https://docs.spring.io/spring-modulith/reference/events.html).
- **OpenJDK:** [JEP 444](https://openjdk.org/jeps/444) использован для различия virtual threads, ограничения ресурсов и CPU parallelism. Особенности pinning проверяются под конкретный JDK.

## Изменения, которые особенно влияют на поведение агента

1. Сначала существующий проект и его совместимость; никаких автоматических переходов на Boot 4, Java 25, WebFlux или микросервисы.
2. Проверка реальной атомарности: self-invocation, rollback rules, независимые транзакции, границы async и внешних эффектов.
3. Retry не считается безопасным без анализа повторяемости операции; неизвестный исход оплаты не превращается в фиктивный успех.
4. Для надёжного DB + event взаимодействия рассматриваются outbox/durable registry, дубликаты, подтверждения и восстановление; нет необоснованного обещания exactly-once.
5. Авторизация проверяется на уровне объекта/tenant; «stateless» не означает автоматическую безопасность от CSRF.
6. Проверки выбираются по свойству: реальные SQL/rollback, отказ downstream, конкурентные записи, контракт API. Пропущенные тесты остаются непроверенными.
7. Наблюдаемость учитывает cardinality, перегрузку, очереди и корректные probes; ёмкость сервиса не объявляется без измерений.

## Обязательные стандарты пользователя

Уточнены **23 октября 2026 года**. Это требования пользователя к работе скилла, а не вывод, что один стек подходит всем проектам:

- Hibernate через JPA — основной доступ к реляционным данным приложения. Прямой SQL допустим для конкретной критичной операции при измеримой необходимости оптимизации; обычный CRUD остаётся на ORM. JPQL/HQL и Criteria относятся к ORM; SQL-миграции и диагностика не подпадают под это ограничение. [Hibernate: способы запросов и native SQL](https://docs.hibernate.org/orm/7.1/introduction/html_single/#queries) подтверждает возможности API, но не устанавливает такую политику выбора.
- Отступы Java-кода выполняются настоящими символами табуляции (`U+0009`), по одному на уровень блока. Это требование пользователя заменяет прежние правила о пробелах и отступе в два пробела, а также рекомендацию выбирать форматтер, который навязывает такие отступы. Остальные совместимые правила [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) сохраняются; полные осмысленные имена переменных, классов и методов по-прежнему обязательны.
- Все репозиторные интерфейсы располагаются в пакете `repository` или его подпакетах по бизнес-смыслу. Это относится и к Spring Data, и к доменным контрактам. Для раздельных архитектурных границ допустимы `domain.repository` и `infrastructure.repository`: название пакета не меняет направление зависимостей.
- Каждый написанный вручную метод должен помещаться на одном стандартном экране редактора. Если в проекте нет принятого определения, рабочее значение скилла — максимум 40 строк от сигнатуры до закрывающей скобки, включая пустые строки и комментарии; более строгий лимит проекта сохраняется. Длинные методы разбиваются по смыслу на вызываемые вспомогательные методы с тем же ограничением. Прежняя фраза об отсутствии ограничения длины методов удалена; число 40 является выбранным ориентиром скилла, а не утверждением о размере любого экрана.
- Clean Code и Clean Architecture Роберта Мартина — основа читаемости, ответственности компонентов и направления зависимостей во всём приложении. [Авторское описание Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) служит первоисточником для направления зависимостей; конкретное применение к Spring изложено в архитектурном справочнике скилла.
- Lombok используется для максимального безопасного сокращения бойлерплейта. Его [генерация конструкторов](https://projectlombok.org/features/constructor) и [состав `@Data`](https://projectlombok.org/features/Data) сверены с официальной документацией; ограничения для JPA-сущностей сформулированы как инженерные меры сохранения семантики.
- [Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html) — основной web-подход: тонкие контроллеры, сценарии приложения и отдельное представление ответа. MVC сочетается с Clean Architecture и не требует HTML для REST API.

В существующем проекте конкретный конфликт совместимости или обязательного инструментария явно объясняется; локальная задача не превращается в полную миграцию стека. Новые стандарты заменяют прежний запрет на добавление Lombok без отдельной причины и нейтральный выбор persistence/web-стека для новых приложений.

## Как обновлять

При изменении версий сверять официальные compatibility/migration документы с проектом. При пересмотре заимствованных решений повторно читать связанные исходные файлы: ссылки на default branch изменяемы. Популярность и даты push — исторический снимок, не бессрочное основание рекомендации. Этот скилл представляет самостоятельный синтез и критический разбор; сторонние скиллы и их установщики целиком не копировались и не запускались.
