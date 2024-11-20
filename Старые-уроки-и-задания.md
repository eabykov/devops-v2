> Первый совет: начните учить английские слова, для начала вам нужно уметь прочесть и понять информацию

Порядок изучения технологий для DevOps: https://roadmap.sh/devops и комментарии к порядку изучения технологий:

1. Выучить основы bash и python/go (дополнительная информация ниже в `Базовые знания Linux, сетей и bash скриптов`)
2. Из операционных систем советую поставить себе Ubuntu: https://ubuntu.com/desktop или если у вас Windows то установить Ubuntu из магазина Microsoft https://apps.microsoft.com/detail/9nz3klhxdjp5
   > Большинство серверов работают на базе linux (как и Ubuntu) который будет удобен в обычном пользовании и подготовке к работе
3. Из текстовых редакторов научиться открывать файлы и редактировать в `nano`, а также в `vim` (если удобно)
4. По терминалу вся информация будет в `Базовые знания Linux, сетей и bash скриптов`
5. По системе контроля версий git ниже в `Git система контроля версий и автоматизация с помощью GitLab CI/CD`
6. По контейнеризации изучать Docker, ниже в `Контейнеризация с помощью Docker и Kubernetes`
7. Насчет Облачных провайдеров, в России лучше использовать Yandex Cloud, SberCloud или VK Cloud
8. Сети изучать ниже в `Базовые знания Linux, сетей и bash скриптов`
9. Изучать Terraform и Ansible как IaC (инфраструктуру как код, ниже в `IaC - (Инфраструктура как код) создание серверов с помощью Terraform и настройка с помощью Ansible`)
10. CI/CD проще изучать на основе GitHub Actions, а потом GitLab CI/CD (автоматизация)
11. Мониторинг ниже в пункте `Мониторинг, золотые сигналы и логгирование`
12. Управление паролями, секретными данными или ключами с помощью Vault
13. Мониторинг приложений: Jaeger Tracing, Sentry
14. Управление логами Graylog и Loki, тоже ниже в `Мониторинг, золотые сигналы и логгирование`
15. Под Kubenretes тоже отдельный пункт ниже совместо с Docker (в последних версиях Kubenretes отказались от Docker)
16. Хранение собраных приложений, docker образов, helm чартов: Nexus и Artifactory
17. Наблюдаемость и управление сетевым трафиком в Kubenretes: istio или linkerd (ниже в пункте с Kubenretes)

Полезные youtube каналы:
- Канал с плейлистами по которым учились многие devops инженеры: https://www.youtube.com/@ADV-IT/playlists
- Канал от создателей платных курсов Merion Academy на котором рассказывают про технологии из RoadMap выше простыми словами: https://www.youtube.com/watch?v=NtGN7Nz6I0c
- Хороший канал по devops, все видео смотреть на скорости 1.25 (если не хотите заснуть): https://www.youtube.com/@pavlenkoat/playlists

<details>
  <summary>Базовые знания Linux, сетей и bash скриптов</summary>

| Цель и что нужно для изучения + Задания | Как поймем что цель достигнута? |
|-|-
| Linux - ядро операционной системы, например Ubuntu построена на базе Linux. Пройти курс https://youtu.be/wdaHKwvNRuU?si=UnvTogPjiVOE5PEc , https://habr.com/ru/articles/788970/ и сделать все задания https://github.com/eabykov/devops-linux | Может устанавливать программы, знает основные команды и может их применять, что такое ядро linux, знает какие основные папки есть в `/`, отвечает на вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/linux и https://github.com/bregman-arie/devops-exercises#operating-system---self-assessment
| Linux скрипты - простые сценарии, автоматизация рутинных задач. Задание: сделать скрипты для всех заданий https://github.com/eabykov/devops-linux и задания 2,5,9 в https://github.com/bregman-arie/devops-exercises/tree/master/topics/shell | Умеет создавать и использовать переменные, может применять условный оператор IF и использовать CASE, умеет использовать циклы, вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/shell
| Сети и сетевые технологии - как сервера (настольные ПК и тд) обьединяются в общую сеть для обмена информацией, пример интернет, глобальная сеть обьединяющая компьютеры по всему миру. Прочесть статью https://habr.com/ru/post/326574/ , https://ru.wikipedia.org/wiki/Маска_подсети и https://habr.com/ru/post/711578/ , посмотреть про websocket https://youtu.be/19d4AXt3dSI | Как подключиться по SSH и как работает SSH, что такое 'пакет', уровни TCP/IP, что такое DNS, что такое HTTP/HTTPS протокол и REST API, что такое IP и маска подсети, как на linux посмотреть сетевые интерфейсы, сниффинг трафика, что такое Nginx (как выглядит конфиг) и round-robin балансировка, вопросы https://github.com/bregman-arie/devops-exercises#network и https://github.com/bregman-arie/devops-exercises/tree/master/topics/dns

</details>

<details>
  <summary>Git система контроля версий и автоматизация с помощью GitLab CI/CD</summary>

| Цель и что нужно для изучения + Задания | Как поймем что цель достигнута?
|-|-|
| Git - система управления версиями для совместной работы над проектом и в случае чего удобному восстановлению к более старой версии https://youtu.be/EeARyFrZsnU . Пройти курс https://www.youtube.com/watch?list=PLg5SS_4L6LYstwxTEOU05E0URTHnbtA0l до 15 урока и создать свой репозиторий на github с несколькими ветками и тегами | Знает что такое commit и как его делать, умеет делать branch и tag и знает в чем между ними разница, знает что такое merge и как исправлять конфликты, знает как откатиться на более старую версию, как склонировать репозиторий локально и как загрузить свои изменения в github, в чем разница межу fetch и pull, вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/git
| CI/CD - выполнение автоматически действий по триггеру, например commit в master, создание merge, создание tag или cron расписанию. В курсе https://www.youtube.com/watch?list=PLg5SS_4L6LYstwxTEOU05E0URTHnbtA0l 15й и 16й, https://youtu.be/tE3u1LquFcg?t=212 скорость 1.25, https://github.com/gitlabhq/gitlabhq/blob/master/doc/ci/docker/using_kaniko.md `.gitlab-ci.yml` как собирать Docker Image в GitLab правильно | Сделал автоматическую сборку своего Docker Image и отправку dockerhub хранилище образов (хранилище образов называют registry), GitLab CI/CD основные понятия, из каких шагов состоит идеальный CI/CD пайплайн, вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/cicd

</details>

<details>
  <summary>Контейнеризация с помощью Docker и Kubernetes</summary>

| Цель и что нужно для изучения + Задания | Как поймем что цель достигнута?
|-|-|
| Docker - упаковка приложения в image в котором будет все что нужно для запуска https://youtu.be/aZTL2zRmOnA и https://habr.com/ru/companies/flant/articles/787494/ | Понимает зачем нужен docker, умеет создавать свой образ и пушить его в dockerhub, умеет запускать несколько образов вместе используя compose, вопросы https://habr.com/ru/company/southbridge/blog/528206/
| Kubernetes - приводит состояние кластера из пункта А в пункт С, нужно только обьяснить с помощью yaml манифестов чего хотим в пункте С. Пройти курс https://learn.microsoft.com/ru-ru/training/modules/intro-to-kubernetes/ вместе с https://github.com/eabykov/kubernetes , https://habr.com/ru/articles/777728/ и поставить локально istio (посмотреть какие сервисы взаимодействуют, есть ли ошибки, сколько запросов в секунду) | Понимает зачем нужен Kubernetes, как устанавливать приложения через helm, вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/kubernetes

Примерный порядок изучения технологии:
- Docker: https://roadmap.sh/docker
- Kubernetes: https://roadmap.sh/kubernetes

</details>

<details>
  <summary>Мониторинг, золотые сигналы и логгирование</summary>

| Цель и что нужно для изучения + Задания | Как поймем что цель достигнута?
|-|-|
| Мониторинг - сбор исторических данных о нашей системе https://youtu.be/wDan20_WyNg использовать пример https://github.com/ruanbekker/docker-monitoring-stack-gpnc , Linux серверах, показателей приложений и их логов, сетевых метрик, и оповещение, если что-то пошло не так. Prometheus + Grafana + любые экспортеры, ELK стек https://youtu.be/ZcC3BTChCY0?t=110 и https://github.com/docker/awesome-compose/tree/master/elasticsearch-logstash-kibana , Трейсинг https://youtu.be/7Dyf4AiUAcQ | Понимает как создавать алерты (оповещения), может настроить мониторинг Docker, linux host, в Kubernetes установить https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack , вопросы https://github.com/bregman-arie/devops-exercises#prometheus , https://github.com/bregman-arie/devops-exercises#monitoring и https://github.com/bregman-arie/devops-exercises#elastic

Логгирование в Kubernetes: https://kubernetes.io/docs/concepts/cluster-administration/logging/

Золотые сигналы: https://habr.com/ru/companies/southbridge/articles/688082/
- Процент успешных запросов
- RPS (requests per second) - количество запросов в секунду
- время обработки запросов

Автоматический мониторинг золотых сигналов (Golden metrics) с помощью Linkerd: https://linkerd.io/2.13/features/telemetry/

Пример алерта в AlertManager который срабатывает по условию из Prometheus

```yaml
- name: Host out of memory
  # description - описание алерта, более детальное пояснение о чем он
  description: Node memory is filling up (< 10% left)
  # query - запрос в случае которого будет срабатывать алерт, правило (например памяти меньше 10%)
  query: '(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 10) * on(instance) group_left (nodename) node_uname_info{nodename=~".+"}'
  # severity - уровень алертинга, важность, например warning это предупреждение, а critical критично и опасно - самый большой приоритет
  severity: warning
  # for - сколько минут должно соблюдаться 'query' выше, 0m сработает алерт сразу, 2m правило в query должно соблюдаться 2 минуты
  for: 2m
```
> Готовые алерты брать тут: https://github.com/samber/awesome-prometheus-alerts/blob/master/_data/rules.yml

Примеры работы с логами в ELK: 
- Приложения database записывает логи на диск в файл: `2023-10-11 18:45:01 INFO Application ready`
- Приложения web-app записывает логи на диск в файл: `18:45:02 2023-10-11 error: failed to start`

Логи которые попадут в Elasticsearch (формат `JSON`):
```json
{"program_name": "database", "date": "2023-10-11", "time": "18:45:01", "level": "INFO", "message": "Application ready"}
{"program_name": "web-app", "date": "2023-10-11", "time": "18:45:02", "level": "ERROR", "message": "failed to start"}
```

</details>

<details>
  <summary>IaC - (Инфраструктура как код) создание серверов с помощью Terraform и настройка с помощью Ansible</summary>

| Цель и что нужно для изучения + Задания | Как поймем что цель достигнута?
|-|-|
| Ansible - упарвляет конфигурацией хостов по SSH https://youtu.be/23Zec3ORJOY . Пройти курс https://www.youtube.com/watch?list=PLg5SS_4L6LYufspdPupdynbMQTBnZd31N 1,6,10,12,14,15,19 не по названию | Понимает зачем нужен Ansible, что такое идемпотентность, что такое playbook, умеет писать свои роли, вопросы https://github.com/bregman-arie/devops-exercises/tree/master/topics/ansible
| Terraform https://youtu.be/ph4iNA0Uuko . Пройти курс https://www.youtube.com/watch?list=PLg5SS_4L6LYujWDTYb-Zbofdl44Jxb2l8 1,3,6,7,12,14,16,18 не по названию | Понимает зачем нужен Terraform, знает как создавать ресурсы (например виртуальную машину), где хранится состояние (информация) о том что сделал terraform, вопросы https://habr.com/ru/company/southbridge/blog/528206/

Генерация пары SSH ключей:
```sh
ssh-keygen -t ed25519 -C 'ВАШ АДРЕС ЭЛЕКТРОННОЙ ПОЧТЫ'
```

Файловая структура Ansible role:
```
roles/                      # роли хранятся в папке 'roles'
    prometheus/             # имя роли, в нашем случае 'prometheus'
        tasks/
            main.yml        #  ниже пример файла
        files/
            prometheus.conf #  файл который будет скопирован на удаленную машину
        vars/
            main.yml        #  переменные для данной роли, например версия prometheus
        defaults/
            main.yml        #  самые базовые переменные по умолчанию, имеют ниже приоритет чем 'vars' выше
```

Пример файла `roles/prometheus/tasks/main.yml`:

```yaml
- name: Install prometheus # устанавливаем prometheus на удаленном сервере
  ansible.builtin.apt:
    name: prometheus       # какую программу будет устанавливать

- name: Copy prometheus config to remote hosts # имя задачи, чтобы мы понимали что делает
  ansible.builtin.copy:
    src: prometheus.conf                  # где файл лежит у нас на компе
    dest: /etc/prometheus/prometheus.conf # то куда файл попадет на удаленных хостах
    owner: aider                          # кто будет хозяином файла на удаленной машине
    group: aider                          # какая группа будет у файла на удаленной машине
    mode: u=rw,g=r,o=r                    # какие права будут у файла на удаленной машине

- name: restart prometheus # перезапустить prometheus
  service:
    name: prometheus  # с каким сервисом будем работать
    state: restarted  # перезапустить prometheus
    enabled: yes      # запускать nginx при перезапуске системы
```

</details>

<details>
  <summary>Вопросы на собеседовании</summary>

Примеры вопросов:
- https://habr.com/ru/articles/775560/

### Linux

1. Что такое systemd
   1. Где находится конфигурация
   2. Какие основные поля в конфигурации
2. Какие есть kill сигналы?
   1. Когда мы во время выполнения команды жмем Ctrl + C то какой сигнал отправляется?
4. Что такое ядро Linux?
   1. Как посмотреть веросию ядра Linux?
   2. Что такое системные и вызовы и какие бывают?
5. Какой командой посмотреть сетевые интерфейсы?
6. Какой командой посмотреть какие приложения занимают те или иные порты?
7. Как забрать права на доступ к файлу или директории в linux?
8. Как можно запланировать выполнение комманды по расписанию, например каждую минуту?
   1. Как помотреть список уже запланированных заданий?
9. Что такое SSH?
   1. Что нужно чтобы подключиться к удаленному серверу через SSH?
   2. Как посмотреть запущен ли SSH сервер на linux хосте?
   3. Где находится конфигурация SSH сервера?
   4. Где хранятся ssh ключи текущего пользователя?
10. Как установить программму в Linux Ubuntu?
   1. Как обновить все программы?
   2. Как посмотреть информацию о комманде или программе?
11. Как заменить одно слово на другое в файле?
12. Как редактировать файл в Linux?
    1. Что такое файловый дискриптор?
13. Как посмотреть запущенные процессы?
14. Как завершить запущенный процесс grafana?

### Git

1. Как создать новую ветку?
   1. Как переключиться на другую ветку?
2. Как откатиться на несколько версий назад чтобы последние изменения ищезли?
3. Как отправить наши текущие изменения в удаленный репозиторий?
4. Как выполнить добавление изменений из одной ветки в другую?
   1. Что делать если при этом возник конфликт?
5. Как скачать себе локально последние изменения из удаленного репозитория (репозиторий уже есть на компьютере)?
6. Чем отличается tag от branch?
7. Чем отличается fetch от pull?

### GitLab

1. Что такое GitLab CI/CD и CI/CD в целом?
   1. Какие поля есть в шагах (stages)?
   2. Как хранить пароли?
   3. Как сделать так чтобы два или больше шагов запускались одновременно?
3. Что такое артефакты и где они хранятся
4. Что такое stage, preprod, prod окружения?
5. Как склонировать себе локально git репозиторий из gitlab используя ssh?

### Сеть и сетевые технологии

1. Что такое прокси и чем отличается от VPN?
2. Что такое балансировщик нагрузки?
3. Что такое кэширующий сервер и какие проблемы решает?
4. Что такое выделенный IP адресс в cloud?
   1. Как правильно ограничить трафик к нашему приложению чтобы оно было доступно только нам?
5. Какие есть уровни модели TCP/IP и пример протоколов на каждом из них?
   1. В чем разница между TCP и UDP?
      1. Что такое трехстороннее рукопожатие?
   3. В чем разница между HTTP и HTTPS?
      1. Опишите жизненный цикл запроса HTTP
      2. Какие существуют методы HTTP?
      3. Какие существуют коды/статусы ответа HTTP?
      4. Какие распространненые заголовки HTTP?
   5. Что такое TLS и как работает?
   6. Что такое SMTP и как работает?
   7. В чем разница между IP и MAC адресами? Для чего они используется?
   8. Какие основные виды HTTP запросов существуют?
6. Что такое TTL (Time to Live)?
7. Как работает DHCP?
8. Что такое DNS сервер?
   1. По какому протоколу работает?
   2. Что должен делать если не нашел запись у себя в конфигурации?

### Docker и работа с упакованными в контейнер приложениями

1. Чем контейнеризация Docker отличается от виртуализаии?
2. Чем отличается контейнер от образа (image)?
3. Как создать свой образ docker?
   1. Что такое базовый образ?
   2. Чем хорошо образ alpine linux и чем он отличается например от образа ubuntu linux?
   3. В чем отличия между COPY и ADD?
   4. Есть ли отличия между CMD и ENTRYPOINT и можно ли их использовать вместе?
   5. Где лучше хранить собраные образы docker?
4. Как запустить несколько образов вместе на своем компьютере для тестирования?
   1. Можно ли в docker compose ограничить использование RAM и CPU для отдельных контейнеров?
   2. Сохранятся ли данные записанные приложением в контенйнере на диск после его перезапуска?
   3. Что такое volumes и для каких приложений использовать их нужно?
   4. Как настроить контейнер так чтобы он перезапускался сам если приложение внутри сломается?
5. Где хранятся volumes и logs в docker?

### Облачные технологии

1. Для чего нам вообще нужны облака?
2. Что такое SaaS, PaaS и IaaS, в чем между ними разница?
3. Что такое VPC, Security Group и EIP например в AWS?

### Ansible и Terraform (инфраструктура как код)

1. VPC, виртуальные машины или например EIP каким бы инструментом создавал?
2. Устанавливал бы docker на linux машину, обновлял сервис, давал права на директорию?

### Kubernetes или коротко k8s

1. Какие компоненты должны быть установлены на master node?
   1. А какие на worker node?
2. Что такое etcd?
   1. Какой тип у этой базы данных?
   2. Почему в etcd должно быть нечетное количество реплик?
3. Как настраивается сеть в k8s?
   1. Что такое CNI?
   2. Каким образом каждому pod выдается отдельный IP?
4. Что такое service?
   1. Как с помощью service обратиться к pod в другом namespace?
   2. Является ли service DNS именем?
   3. По какому правилу service будет распределять трафик между pod?
   4. Чем отличаются service вида headless и clusterIP?
5. Какие виды prob вы знаете?
   1. Для чего нужна каждая из них?
6. Сколько контейнеров может быть в одном pod?
   1. Нужно ли создавать service для того чтобы контейнеры отправляли друг другу запросы в рамках одного pod?
7. Какие ingress контроллеры вы знаете?
   1. Что такое ingress, какие ресурсы он связывает на сетевом уровне?
   2. Как в ingress использовать SSL сертификаты?
8. Чем отличаются annotations от labels и приведите по одному примеру использования?
9. Основные различия между Deployment, StatefulSet и DaemonSet?
10. Что такое lifecycle хуки?
    1. Для чего используется preStop хук с `sleep 10` функцией?
11. Для чего нужны лимиты (limits) и запросы (requests) у pod?
    1. HPA при расчетах использует limits или requests?
    2. Планировщик при выборе на какую node размещать pod учитывает limits?
12. Каким образом можем ограничить права пользователей в k8s?
13. Чем так удобны helm charts?
   1. Какие основные компоненты чарта?
   2. Что находится в файле `values.yaml` и `Chart.yaml`?
   3. Для чего в папке templates создают файл `_helpers.tpl`?
   4. Как создать цикл который создаст несколько сущностей (например ports в service)?
   5. Как сделать условный оператор для boolean значений и для строк?
   6. Что будет записано вместо темплейта `{{ divf .Values.replicaCount .Values.zones | ceil }}`? Как может пригодиться читать тут https://github.com/eabykov/devops-kubernetes/blob/main/ЛУЧШИЕ_ПРАКТИКИ.md в коментарие к разделу *Распределите ваши pod по разным node и разным зонам (датацентрам)*

### Vault

1. Для чего нужен Vault?
2. Что такое KV secrets engine в vault?
3. Как сделать так чтобы в Kubernetes использовать например для Deployment секреты из Vault?

### Сервисная сетка (service mesh)

### Jaeger трейсинг запросов

1. Что нужно сделать в приложении чтобы его трейсы появились в Jaeger?

### Flux и Flagger (канареечные релизы)

</details>
