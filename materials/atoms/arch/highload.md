# === ТЕСТИРОВАНИЕ ВЫСОКИХ НАГРУЗОК ===

> что такое WireMock

WireMock — это инструмент (Java-библиотека и standalone HTTP-сервер) для
создания заглушек (моков) и эмуляции HTTP/REST API. [1]
(https://habr.com/ru/companies/rostelecom/articles/679276/), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup), [3]
(https://habr.com/ru/articles/945740/)

Для чего используется

• Изоляция тестов: Позволяет тестировать микросервисы и приложения без обращения
к реальным внешним сервисам, базам данных или сторонним API. [1]
(https://www.reddit.com/r/dotnet/comments/1fttskt/how_to_test_integrations_with_apis_using_wiremock/?tl=ru), [2]
(https://habr.com/ru/articles/945740/), [3]
(https://afkoffer.com/interview/qa/qa-wiremock-setup)
• Разработка в условиях неопределенности: Дает возможность вести разработку и
UI/автотесты, когда нужный бэкенд-сервис еще не готов или находится в стадии
проектирования. [1] (https://habr.com/ru/articles/945740/)
• Эмуляция ошибок и задержек: Позволяет проверять реакцию системы на
нестандартные ситуации — таймауты, ответы с ошибками (например, HTTP 400, 500)
или поврежденное тело ответа. [1]
(https://habr.com/ru/companies/rostelecom/articles/679330/), [2]
(https://ru.linkedin.com/posts/zhuravlevod_wiremock-%D0%BB%D0%B5%D0%B3%D0%BA%D0%B8%D0%B9-%D0%BF%D1%83%D1%82%D1%8C-%D0%BA-%D1%8D%D1%84%D1%84%D0%B5%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D0%BC%D1%83-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E-activity-7402260333962166272-EWvR)
• Запись и воспроизведение трафика (Record & Playback): Может работать как
прокси-сервер, перехватывая реальные запросы и ответы для их последующего
использования в качестве стабов. [1]
(https://habr.com/ru/companies/uzum/articles/959736/), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup)

Основные возможности

• Сопоставление запросов (Request Matching): Гибкая настройка правил
срабатывания заглушек по HTTP-методам, URL (включая регулярные выражения),
заголовкам, параметрам (query-params) и телу запроса. [1]
(https://ru.linkedin.com/posts/zhuravlevod_wiremock-%D0%BB%D0%B5%D0%B3%D0%BA%D0%B8%D0%B9-%D0%BF%D1%83%D1%82%D1%8C-%D0%BA-%D1%8D%D1%84%D1%84%D0%B5%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D0%BC%D1%83-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E-activity-7402260333962166272-EWvR), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup)
• Сценарии (Stateful Mocks): Поддержка состояний (state machine), когда один и
тот же эндпоинт может возвращать разные ответы при последовательных вызовах
(например, первый запрос дает ошибку, а повторный — успех). [1]
(https://habr.com/ru/companies/uzum/articles/959736/), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup), [3]
(https://ru.linkedin.com/posts/zhuravlevod_wiremock-%D0%BB%D0%B5%D0%B3%D0%BA%D0%B8%D0%B9-%D0%BF%D1%83%D1%82%D1%8C-%D0%BA-%D1%8D%D1%84%D1%84%D0%B5%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D0%BC%D1%83-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E-activity-7402260333962166272-EWvR)
• Динамические ответы (Response Templating): Использование шаблонизатора для
генерации случайных чисел, дат, UUID или извлечения данных из входящего
запроса. [1]
(https://software-testing.ru/library/testing/testing-automation/3896-api-mocking-in-csharp), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup), [3]
(https://habr.com/ru/companies/rostelecom/articles/679330/)
• Встраивание и запуск: Может использоваться как библиотека внутри
unit/integration-тестов на Java (.NET через WireMock.Net) или запускаться
отдельно в виде автономного сервера. [1]
(https://habr.com/ru/companies/rostelecom/articles/679276/), [2]
(https://afkoffer.com/interview/qa/qa-wiremock-setup), [3]
(https://software-testing.ru/library/testing/testing-automation/3896-api-mocking-in-csharp)
Хотите узнать, как написать первый стаб в WireMock или как интегрировать его в
ваши автотесты?


> это может быть прям отдельный микросервис?

Да, WireMock может работать как полноценный отдельный микросервис. Для этого его
запускают в так называемом автономном режиме (Standalone mode).
В такой архитектуре WireMock разворачивается в вашем окружении (например, в
Kubernetes или Docker-контейнере) как независимый компонент системы. Другие
микросервисы общаются с ним по сети точно так же, как с реальным сторонним API.

Как устроен WireMock как отдельный микросервис:

• Запуск через Docker: Это самый популярный способ. Существует официальный
Docker-образ (wiremock/wiremock), который можно одной командой поднять в вашей
сети.
• Настройка без кода (JSON-маппинги): Обычному микросервису нужен программный
код, а WireMock-сервису достаточно конфигурационных файлов. Вы просто создаете
JSON-файлы с правилами (какой запрос пришел — какой ответ вернуть) и кладете их
в специальную папку mappings. WireMock подхватывает их на лету.
• Управление через API: У WireMock-микросервиса есть свой собственный
административный API (/__admin/). Ваши автотесты или другие сервисы могут "на
лету" отправлять туда HTTP-запросы, чтобы создавать новые заглушки, удалять
старые или проверять, поступали ли на сервер определенные вызовы.
• Использование в CI/CD: Такой микросервис часто поднимают в тестовом контуре
(Stage/QA) на время прохождения интеграционных или нагрузочных тестов, чтобы не
зависеть от стабильности внешних систем.
Хотите посмотреть пример Docker Compose файла для поднятия такого микросервиса
или шаблон JSON-конфигурации для заглушки?

> не, лучше скажи, можно с его помощью проверить большое количество запросов на
> мой микросервис?

Да, но с важной оговоркой: WireMock используется не для генерации нагрузки, а
для того, чтобы ваш микросервис эту нагрузку выдержал. [1]
(https://www.wiremock.io/post/running-effective-load-tests-using-mocks-and-simulated-environments)
Сам WireMock не умеет засыпать ваш сервис миллионами запросов — для этого нужны
специальные инструменты (JMeter, Gatling, k6, Locust). Однако WireMock незаменим
в качестве стабильной заглушки для внешних зависимостей во время таких
нагрузочных тестов. [1]
(https://www.skillsoft.com/course/rest-assured-techniques-using-wiremock-api-mocking-jmeter-load-testing-020e9f5a-292a-4bd0-b41e-2818d4bde5af), [2]
(https://habr.com/ru/companies/serverspace/articles/690578/), [3]
(https://www.wiremock.io/post/running-effective-load-tests-using-mocks-and-simulated-environments), [4]
(https://blog.dtdl.in/use-case-of-wiremock-where-we-can-use-wiremock-load-testing-mock-testing-d92bcd1942ad)

Как это работает на практике

Когда вы проверяете свой микросервис под высокой нагрузкой (например, 2000
запросов в секунду), он, скорее всего, ходит в другие системы: платежные шлюзы,
CRM, SMS-сервисы.
Если тестировать всё вместе, чужие тестовые API начнут тормозить, заблокируют
вас за спам или упадут, что испортит ваши графики. [1]
(https://www.wiremock.io/post/running-effective-load-tests-using-mocks-and-simulated-environments)
Вместо этого вы:

1. Перенаправляете все исходящие вызовы своего микросервиса на
   WireMock-микросервис.
2. С помощью генератора нагрузки (например, JMeter) бьете по своему
   микросервису.
3. Ваш сервис обрабатывает поток запросов и мгновенно получает ответы от
   WireMock, позволяя вам измерить чистую производительность именно вашего
   кода. [1]
   (https://www.wiremock.io/post/running-effective-load-tests-using-mocks-and-simulated-environments), [2]
   (https://www.skillsoft.com/course/rest-assured-techniques-using-wiremock-api-mocking-jmeter-load-testing-020e9f5a-292a-4bd0-b41e-2818d4bde5af)

Особенности настройки WireMock для высокой нагрузки (High Load)

Из коробки WireMock оптимизирован для функциональных тестов, а не для
гигантского трафика. Чтобы одиночный инстанс WireMock держал от 1 000 до 10 000+
запросов в секунду, его нужно запускать со специальными флагами: [1]
(https://www.cecg.io/blog/scaling-an-http-stub-for-load-testing)
• --no-request-journal — Самое важное. По умолчанию WireMock записывает в память
(Heap) логи абсолютно всех входящих запросов, чтобы вы могли их проверить. Под
нагрузкой память забьется за пару минут, и сервер упадет с ошибкой OutOfMemory.
Этот флаг отключает логирование.
• --async-response-enabled=true — включает асинхронную обработку запросов, что
резко увеличивает пропускную способность (Throughput).
• --container-threads — позволяет увеличить количество потоков для обработки
входящих подключений (по умолчанию их всего 10). [1]
(https://stackoverflow.com/questions/44792229/wiremock-performance-and-load-testing), [2]
(https://github.com/Rbillon59/wiremock-loadtest), [3]
(https://wiremock.org/docs/configuration/)
Если мощности одного инстанса все равно не хватает, WireMock без проблем
горизонтально масштабируется в Kubernetes (через стандартный Deployment и
ReplicaSet). [1]
(https://www.cecg.io/blog/scaling-an-http-stub-for-load-testing)
Вам подсказать, как лучше настроить связку для нагрузочного тестирования
(например, JMeter + ваш сервис + WireMock) или у вас асинхронное взаимодействие
(через Kafka/RabbitMQ), где логика тестов немного отличается?


> у меня кафка + kubernetes + 6 микросервисов. я пытаюсь понять как можно
> протестировать мои микросервисы на нагрузки. хотя бы в общих чертах
>

Тестирование архитектуры из 6 микросервисов, связанных через Kafka и развернутых
в Kubernetes, отличается от тестирования классических REST-приложений. Главная
особенность здесь — асинхронность. [1]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/), [2]
(https://purpleschool.ru/knowledge-base/microservices/integration/kafka-mikroservisy)
В общих чертах стратегия тестирования такой системы строится на трех китах:
генерация входящего потока, изоляция внешнего мира и сбор метрик. [1]
(https://habr.com/ru/companies/otus/articles/994142/), [2]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)

Шаг 1. Определить точки входа нагрузки

Вам нужно понять, как именно «закачивается» работа в вашу систему. Обычно бывает
два сценария:

1. Синхронный вход (HTTP/gRPC): Пользователи бьют в API-Gateway (первый
   микросервис), а он уже генерирует события в Kafka. В этом случае инструмент
   нагрузки имитирует действия пользователей. [1]
   (https://habr.com/ru/companies/otus/articles/958840/)
2. Асинхронный вход (Событийный): Нагрузка начинается сразу с Kafka (например,
   данные льются от внешних партнеров или IoT-устройств). В этом случае
   инструмент должен уметь напрямую писать сообщения в стартовый топик с
   огромной скоростью. [1]
   (https://purpleschool.ru/knowledge-base/microservices/integration/kafka-mikroservisy), [2]
   (https://habr.com/ru/companies/reksoft/articles/900918/)

Шаг 2. Выбрать инструменты

Для этой архитектуры вам понадобятся три категории инструментов:

1. Генератор нагрузки:
   • Если вход через HTTP: отлично подойдут k6 (пишется на JavaScript, очень
   легкий для K8s) или Gatling.
   • Если нужно засыпать сообщениями саму Kafka: Pepper-Box (плагин для JMeter)
   или встроенный инструмент kafka-producer-perf-test.sh, либо скрипты на k6 со
   специальным расширением для Kafka. [1]
   (https://www.reddit.com/r/kubernetes/comments/tz3720/kubernetes_load_testing_8_tools_best_practices/?tl=ru), [2]
   (https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)
2. Заглушки (Те самые моки):
   • Если ваши 6 микросервисов ходят за данными наружу (в сторонние банки, CRM,
   СМС-шлюзы), используйте WireMock Standalone (как отдельный под в Kubernetes),
   настроенный в High-Load режиме (с флагом --no-request-journal).
3. Мониторинг (Глаза системы): Без него тест бесполезен. Вам нужна связка
   Prometheus + Grafana, чтобы видеть утилизацию ресурсов. [1]
   (https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)

Шаг 3. Что именно проверять (Специфика Kafka + K8s)

В отличие от обычных сайтов, где меряют только "время ответа" (Latency), в вашей
архитектуре ключевыми будут другие метрики: [1]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/), [2]
(https://habr.com/ru/companies/otus/articles/840402/)
• Consumer Lag (Отставание консьюмеров): Это самый главный показатель.
Показывает, успевают ли ваши микросервисы вычитывать сообщения из Kafka так же
быстро, как они туда поступают. Если лаг постоянно растет — сервис не
справляется. [1] (https://habr.com/ru/companies/reksoft/articles/900918/)
• Поведение Kubernetes HPA (Horizontal Pod Autoscaler): Если у вас настроено
автомасштабирование подов, тест покажет, успевает ли K8s поднять новые копии
микросервисов до того, как упадет брокер или накопится критический лаг в
Kafka. [1]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)
• Лимиты ресурсов (CPU/Memory): В Kubernetes микросервисы зажаты в limits. Часто
бывает, что при нагрузке сервис упирается в лимит по памяти, и K8s убивает его
по ошибке OOMKilled. Тест поможет найти правильный баланс ресурсов. [1]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)
• Партиционирование Kafka (Секционирование): Вы увидите, эффективно ли
распределяется нагрузка. Если у топика 3 партиции, а вы масштабировали
микросервис до 6 подов — 3 пода будут просто простаивать (так как одна партиция
читается только одним консьюмером в группе). [1]
(https://purpleschool.ru/knowledge-base/microservices/integration/kafka-mikroservisy), [2]
(https://habr.com/ru/companies/otus/articles/840402/)

Шаг 4. Порядок проведения тестов

Не пытайтесь сразу включить всё на максимум. Действуйте поэтапно: [1]
(https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/)

1. Тест стабильности (Baseline): Подайте небольшую нагрузку (например, 50
   запросов в секунду) на 1 час. Убедитесь, что лаг в Kafka не растет, память в
   Kubernetes стабильна.
2. Стресс-тест (Stress Testing): Постепенно повышайте нагрузку до тех пор, пока
   что-то не сломается. Ваша цель — найти "узкое горлышко". Что упадет первым?
   База данных микросервиса №3? Забьется CPU на микросервисе №5? Начнет расти
   лаг?
3. Тест на отказоустойчивость (Chaos Engineering): Прямо во время средней
   нагрузки "прибейте" один из подов микросервиса вручную (kubectl delete pod).
   Посмотрите, как система перебалансирует нагрузку на Kafka и возникнут ли
   потери сообщений. [1]
   (https://datafinder.ru/products/kursy/uchebnyy-kurs-sovremennaya-arhitektura-hranilishcha-dannyh/apache-kafka-s-nulya/testirovanie-kafka-arhitektur-nagruzochnoe-testirovanie-integracionnye-testy-chaos-testing), [2]
   (https://doersdoings.ru/technologies/performance-testing-v-epohu-mikroservisov-i-kubernetes/), [3]
   (https://habr.com/ru/companies/otus/articles/994142/)
   Скажите, ваши микросервисы принимают нагрузку изначально через HTTP/REST или
   вам нужно отправлять миллионы сообщений напрямую в Kafka? Могу подсказать, с
   какого именно инструмента (k6 или JMeter) будет проще начать в вашем случае.
