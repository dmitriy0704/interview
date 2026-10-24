> слушай, я тут сейчас разбираюсь балансировкой Loadblancer в Kubernetes и че-то
> туплю. в кластере я насчитал аж три балансера. Я читал, что в продакшене на
> некоторых коммерческих облачных сервисах прям в самом начале кластера выдается
> статический ip адрес и к нему привязывается LoadBalancer, а за ним идет
> например
> ingres (nginx), далее за ingres я могу поставить Spring Cloud Gateway и в нем
> тоже могу настроить балансировщик, потом на сервисе Service же приходит запрос
> и
> он перекидывает запрос на kublet а он уже выбирает поды, куда слать запрос.
> или
> я все перепутал?

Вы абсолютно ничего не перепутали, а разложили всё по полочкам на удивление
точно! В продакшене архитектура трафика выглядит именно так. Ваши «три
балансировщика» — это не ошибка, а три разных уровня абстракции, каждый из
которых решает свою задачу: на уровне провайдера, на уровне кластера и на уровне
приложения.
Давайте проследим путь запроса пользователя с самого начала до пода и разберем,
зачем нужен каждый шаг.

Шаг 1. Облачный балансировщик (Внешний LoadBalancer)

Это самый первый уровень. Вы правы: когда в Kubernetes вы создаете манифест
Service с типом LoadBalancer, облачный провайдер (AWS, Yandex Cloud и т.д.)
выделяет реальный статический IP-адрес снаружи кластера.
• Зачем он нужен: Kubernetes-кластер закрыт от внешнего мира. Этот облачный
балансировщик — «железные» или виртуальные сервера самого дата-центра, которые
принимают трафик из интернета и перенаправляют его внутрь вашей приватной сети
Kubernetes на ноды (сервера) кластера.

Шаг 2. Ingress-контроллер (например, Nginx Ingress)

Трафик от облачного балансировщика прилетает внутрь кластера прямо на поды
Ingress-контроллера.
• Зачем он нужен: Внешний LoadBalancer — «глупый», он умеет балансировать только
по IP и портам (L4-уровень). А Ingress умеет читать HTTP-заголовки, куки и
доменные имена (L7-уровень).
• Его задача: Посмотреть на URL-адрес. Если пользователь пришел на :
//my-site.com, Ingress понимает, в какой именно микросервис внутри кластера
нужно отправить этот запрос.

Шаг 3. API Gateway приложения (Spring Cloud Gateway или Istio / Envoy)

В вашей архитектуре эту роль играет Spring Cloud Gateway (или прокси-сервер
Envoy, если мы говорим в контексте Istio). Трафик из Ingress приходит сюда.
• Зачем он нужен: Ingress — это инструмент системных администраторов, а API
Gateway — это часть вашего Java/Spring-приложения.
• Его задача: Здесь происходит тонкая бизнес-логика. Он проверяет авторизацию
пользователя (JWT-токены), собирает сквозные логи, режет подозрительный трафик
(Rate Limiting) и занимается динамической балансировкой. Например, если вы
проводите A/B тестирование, Spring Cloud Gateway может направить 90% трафика на
старую версию сервиса, а 10% — на новую.

Шаг 4. Внутренний Service в Kubernetes

Когда Spring Cloud Gateway решил отправить запрос, например, в orders-service,
он делает запрос на DNS-имя этого Kubernetes-сервиса.
• Зачем он нужен: Поды в Kubernetes постоянно умирают и рождаются, их IP-адреса
постоянно меняются. Service — это постоянный, бессмертный IP-адрес внутри
кластера.
• Как происходит балансировка здесь: На самом деле Service — это не запущенная
программа, а просто набор правил маршрутизации в сетевом плагине Kubernetes
(через технологию IPVS или iptables на компоненте kube-proxy, который управляет
сетевыми картами нод). Он случайным образом (Round Robin) выбирает из списка
живых подов один IP-адрес и перенаправляет сетевой пакет туда.
• Финал: Пакет прилетает на сетевой интерфейс конкретной ноды кластера, где
kubelet и контейнерный движок (Docker/containerd) передают его внутрь вашего
пода с микросервисом.

Резюме: почему их так много и это нормально?

1. Облачный LoadBalancer впускает трафик из интернета в дата-центр.
2. Ingress (Nginx) сортирует трафик по доменам и путям (маршрутизация).
3. Spring Cloud Gateway (или Istio / Envoy) проверяет безопасность (токены,
   права) и делает "умную" бизнес-балансировку.
4. Kubernetes Service делает финальное распределение по конкретным живым
   контейнерам (подам).
   Вы всё поняли абсолютно верно. Эта многослойность позволяет распределять
   миллионы запросов в секунду (RPS) без сбоев и перегрузок.
   Если у вас в кластере развернут Istio, то функции Ingress и Spring Cloud
   Gateway часто объединяют в один компонент — Istio Ingress Gateway на базе
   Envoy. Хотите узнать, как Service Mesh позволяет выкинуть лишнее звено и
   упростить эту цепочку?

> да, но все же, spring cloud gateway балансирует между несколькими
> экземплярами- подами сервиса и kubernetes service через kublete тоже делает
> балансировку по подам?

Вы наступили на очень важный архитектурный нюанс. На самом деле, в стандартной
конфигурации они не дублируют друг друга, а дополняют, но если настроить систему
неправильно, то действительно возникнет двойная балансировка. [1]
(https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
Давайте разберем два сценария, как Spring Cloud Gateway общается с Kubernetes
Service, и почему во втором случае никакой магии Istio даже не потребуется.

Сценарий 1. Классический (Без библиотек Discovery) — Балансирует только
Kubernetes

Это самый частый случай в Kubernetes. Вы настраиваете Spring Cloud Gateway так,
чтобы он просто перенаправлял запросы на доменное имя Kubernetes Service. [1]
(https://docs.spring.io/spring-cloud-kubernetes/reference/load-balancer.html)
• В коде Gateway прописано: uri: http://orders-service:8080. [1]
(https://docs.spring.io/spring-cloud-kubernetes/reference/load-balancer.html)
• Что делает Gateway: Он вообще не знает, сколько подов у вашего orders-service
и какие у них IP-адреса. Для него существует только один единственный
адрес — http://orders-service. Gateway берет запрос и отправляет его на этот
адрес. Сам Gateway в этом сценарии ничего не балансирует.
• Что делает Kubernetes Service: Запрос прилетает на виртуальный IP-адрес
сервиса. Сетевой плагин Kubernetes (на базе iptables / IPVS) перехватывает этот
пакет и случайным образом (Round Robin) подставляет вместо него IP-адрес одного
из живых подов. [1]
(https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
• Итог: Балансировка происходит один раз силами самого Kubernetes.

Сценарий 2. Client-Side (С библиотекой Spring Cloud Kubernetes) — Балансирует
Gateway

Иногда разработчики хотят, чтобы Spring Cloud Gateway сам управлял балансировкой
(например, использовал хитрые алгоритмы вроде Weighted Response Time — слать
меньше запросов на те поды, которые дольше отвечают). Для этого в проект
добавляют библиотеку spring-cloud-kubernetes-loadbalancer. [1]
(https://docs.spring.io/spring-cloud-kubernetes/reference/load-balancer.html), [2]
(https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
Вот тут и начинается то, о чем вы подумали:

1. Gateway узнает IP-адреса подов: С помощью специального API (DiscoveryClient)
   Gateway обращается напрямую к Kubernetes API и спрашивает: «Дай мне список
   IP-адресов всех подов, которые прячутся за orders-service». [1]
   (https://docs.spring.io/spring-cloud-kubernetes/reference/load-balancer.html)
2. Gateway делает выбор: Внутри Gateway плагин Spring Cloud LoadBalancer
   выбирает из списка конкретный IP пода (например, 10.244.1.45). [1]
   (https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
3. Gateway отправляет запрос: Запрос улетает из Gateway напрямую по IP-адресу
   конкретного пода, минуя общие правила балансировки Kubernetes Service. [1]
   (https://docs.spring.io/spring-cloud-kubernetes/reference/load-balancer.html)
4. Итог: Балансировка снова происходит один раз, но теперь её сделал сам Spring
   Cloud Gateway в своей оперативной памяти. [1]
   (https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)

А при чем тут Istio? (Как Service Mesh убирает эту дилемму)

Если в вашей системе появляется Istio, то и Ingress, и Spring Cloud Gateway
часто становятся лишними звеньями, потому что Istio забирает всю эту логику
себе. [1]
(https://deckhouse.io/products/kubernetes-platform/documentation/v1/user/network/managing_request_between_service_istio.html)
В мире Istio:
• Рядом с вашим Gateway и рядом с каждым подом orders-service сидит прокси Envoy
(Sidecar).
• Когда ваш Gateway пытается отправить запрос на http://orders-service, его
Envoy-сайдкар перехватывает этот запрос.
• Envoy внутри себя имеет актуальный список всех конечных подов (Endpoints),
который он получает от панели управления Istio.
• Envoy сам выбирает конкретный под (делает балансировку L7, mTLS шифрование,
проверяет Circuit Breaker) и отправляет запрос напрямую в нужный под, полностью
игнорируя стандартный механизм балансировки iptables от Kubernetes Service. [1]
(https://deckhouse.io/products/kubernetes-platform/documentation/v1/user/network/managing_request_between_service_istio.html), [2]
(https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
Поэтому в современной архитектуре Kubernetes + Istio разработчики стараются не
использовать умную балансировку внутри Spring Cloud (как в Сценарии 2). Они
оставляют приложения "глупыми", позволяя им слать запросы на обычные адреса
Kubernetes Service, а всю "магию" и "умную" балансировку перекладывают на слой
Istio / Envoy. [1]
(https://deckhouse.io/products/kubernetes-platform/documentation/v1/user/network/managing_request_between_service_istio.html), [2]
(https://medium.com/@logiqpool/%EF%B8%8F-load-balancer-face-off-spring-cloud-vs-kubernetes-who-handles-the-heat-98e849f8224a)
Скажите, у вас сейчас Spring Cloud Gateway настроен по Сценарию 1 (просто
пересылает на URL сервиса) или вы используете Spring Cloud Kubernetes для
обнаружения подов? Можем посмотреть, как лучше адаптировать его под работу с
Istio.

