### [Вернуться к главной странице, списку всех тем](README.md)

## 7. Автоматическое создание, настройка и обновление множества ресурсов в облаке

> **Что ты узнаешь:** как описать серверы, сети и базы текстовыми файлами и разворачивать одинаковые окружения одной командой вместо кликов в консоли
>
> **Что нужно знать заранее:** тема 6 (облако, ВМ, сети), тема 3 (git и PR), тема 1 (SSH, systemd, YAML)
>
> **Сколько времени займёт:** 3 часа теории + 8 часов практики
>
> **Твой шаг в сквозном проекте:** развернёшь всю инфраструктуру «Заметок» с нуля одной командой — и так же одной командой снесёшь её без следа

---

В предыдущей теме мы создавали виртуальные машины, базы и хранилища кликами в облачной консоли или CLI-командами. Это работает, пока у тебя один-два сервера. Но что, если завтра нужно поднять 50 серверов, по 10 баз данных, 20 балансировщиков, и точно так же повторить всё это в трёх окружениях — dev, staging, prod? Кликать руками — это сотни часов работы и гарантированные ошибки.

Представь, что у тебя есть **рецепт пирога**, записанный на бумаге. Ты можешь приготовить по нему пирог дома, можешь дать рецепт другу — он приготовит точно такой же. А можно загрузить рецепт в робота-кондитера — и он будет печь идентичные пироги в любом количестве. Точно так же работает **Infrastructure as Code (IaC)** — инфраструктура как код: ты описываешь нужную инфраструктуру в текстовых файлах, а инструменты автоматически создают её в облаке.

Двумя самыми популярными инструментами IaC являются:
- **Terraform** (и его открытый форк **OpenTofu**) — создаёт ресурсы в облаке: ВМ, сети, базы, кластеры Kubernetes.
- **Ansible** — настраивает уже существующие машины: ставит программы, кладёт конфиги, перезапускает сервисы.

Они дополняют друг друга. Сначала Terraform создаёт «коробку» (саму ВМ), потом Ansible заходит внутрь по SSH и устанавливает там nginx, базу, обновляет пакеты.

### Что такое Infrastructure as Code

IaC — это подход, при котором всю инфраструктуру (серверы, сети, базы, права доступа) описывают в текстовых файлах, хранят в Git, и применяют через автоматизированные инструменты. Главные принципы:

1. **Декларативность** — ты описываешь, *что должно быть* («3 ВМ, 1 балансировщик, 1 БД»), а не *как это сделать пошагово*. Инструмент сам разберётся.
2. **Идемпотентность** — повторное применение одного и того же кода ничего не ломает. Если ВМ уже есть — она не пересоздаётся.
3. **Версионирование** — вся инфраструктура в Git, видна полная история: кто, когда и почему её менял.
4. **Воспроизводимость** — одним и тем же кодом можно развернуть dev, staging, prod, и они будут идентичны.
5. **Code review** — изменения инфраструктуры проходят через Pull Request, как обычный код. Это резко снижает количество ошибок.

### Terraform — создание ресурсов в облаке

**Terraform** — инструмент от компании HashiCorp, который умеет работать с сотнями облачных провайдеров через специальные модули — **providers**: yandex-cloud, aws, azurerm, kubernetes, postgresql, github и т. д.

> ⚠️ **Важный нюанс 2024–2026.** В августе 2023 года HashiCorp поменяла лицензию Terraform с открытой Mozilla Public License на закрытую Business Source License (BSL), что вызвало недовольство сообщества. В результате появился открытый форк — **OpenTofu**, поддерживаемый Linux Foundation. Синтаксис и команды идентичны (`tofu plan`, `tofu apply` вместо `terraform plan`, `terraform apply`), большинство модулей совместимы. В российских компаниях и open-source проектах сейчас активно мигрируют на OpenTofu. Дальше в этой теме мы пишем «Terraform», но всё то же самое относится и к OpenTofu.

#### Основные понятия Terraform

1. **Provider** — модуль для работы с конкретным облаком (например, `yandex` или `aws`). Подключается в коде, после чего Terraform умеет создавать ресурсы этого облака.
2. **Resource** — описание одного ресурса в облаке: ВМ, диска, сети, балансировщика.
3. **Data Source** — способ «прочитать» уже существующий ресурс (например, готовый образ ОС), не создавая его.
4. **Variable** — переменная, чтобы один и тот же код можно было применить с разными значениями (например, `vm_count = 3` для dev, `vm_count = 20` для prod).
5. **Output** — значение, которое Terraform «отдаёт наружу» после применения (например, IP-адрес созданной ВМ).
6. **State (состояние)** — файл `terraform.tfstate`, в котором Terraform запоминает, какие ресурсы он уже создал. Это критически важный файл, без него Terraform «забывает» обо всех созданных ресурсах. Хранить его рекомендуется не локально, а в **remote backend**: S3-совместимом хранилище, Yandex Object Storage, GitLab Managed Terraform State.
7. **Module** — переиспользуемый набор ресурсов. Например, модуль «standard-web-service» создаёт сразу ВМ + балансировщик + DNS-запись.
8. **Plan и Apply** — рабочий цикл: `terraform plan` показывает, что *будет* сделано (только показывает!), `terraform apply` — реально применяет изменения.

#### Жизненный цикл работы с Terraform

```
1. terraform init       — скачать провайдеры, инициализировать
2. terraform fmt        — отформатировать код в стандартном виде
3. terraform validate   — проверить синтаксис
4. terraform plan       — посмотреть, ЧТО будет создано/удалено/изменено
5. terraform apply      — реально применить изменения
6. terraform destroy    — удалить ВСЁ, что создал Terraform (для очистки)
```

#### Пример конфигурации Terraform для Yandex Cloud

```hcl
# Указываем, какие провайдеры используем
terraform {
  required_version = ">= 1.5"
  required_providers {
    yandex = {
      source  = "yandex-cloud/yandex"
      version = "~> 0.130"
    }
  }

  # Храним state не локально, а в облачном хранилище
  backend "s3" {
    endpoints = { s3 = "https://storage.yandexcloud.net" }
    bucket    = "my-terraform-state"
    region    = "ru-central1"
    key       = "prod/terraform.tfstate"

    skip_region_validation      = true
    skip_credentials_validation = true
    skip_requesting_account_id  = true
    skip_s3_checksum            = true
  }
}

# Настройка провайдера
provider "yandex" {
  zone = "ru-central1-a"
}

# Переменные
variable "vm_count" {
  description = "Сколько одинаковых ВМ создавать"
  type        = number
  default     = 1
}

# Поиск свежего образа Ubuntu 24.04 — data source, не создаёт ничего
data "yandex_compute_image" "ubuntu" {
  family = "ubuntu-2404-lts"
}

# Создаём собственную сеть
resource "yandex_vpc_network" "main" {
  name = "main-network"
}

resource "yandex_vpc_subnet" "main" {
  name           = "main-subnet"
  network_id     = yandex_vpc_network.main.id
  zone           = "ru-central1-a"
  v4_cidr_blocks = ["10.0.1.0/24"]
}

# Создаём count копий ВМ
resource "yandex_compute_instance" "web" {
  count = var.vm_count
  name  = "web-${count.index}"

  resources {
    cores         = 2
    memory        = 2
    core_fraction = 20
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.id
      size     = 20
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.main.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
  }
}

# Выводим публичные IP всех созданных машин
output "public_ips" {
  value = yandex_compute_instance.web[*].network_interface.0.nat_ip_address
}
```

После запуска `terraform apply` Terraform создаст в Yandex Cloud сеть, подсеть и одну ВМ. Изменив `vm_count = 5` и снова запустив apply — добавит ещё 4 машины (старая останется, Terraform поймёт, что она уже существует).

### Ansible — настройка машин по SSH

Когда машины созданы, нужно их настроить: установить программы, положить конфиги, открыть порты. Этим занимается **Ansible**.

Главные особенности:

1. **Agentless** — на управляемой машине не нужно ничего устанавливать заранее, кроме SSH-сервера и Python. Ansible сам подключается, выполняет команды и отключается.
2. **Идемпотентность** — Ansible проверяет текущее состояние, и если оно уже соответствует желаемому — ничего не делает. Можно запускать playbook хоть 100 раз — результат одинаковый.
3. **Декларативность через модули** — ты не пишешь `apt install nginx`, ты пишешь «модуль `ansible.builtin.apt` должен обеспечить, чтобы пакет `nginx` был установлен». Ansible сам понимает, что нужно сделать.
4. **YAML-синтаксис** — описание читается даже не-программистами.

#### Основные понятия Ansible

1. **Inventory** — файл со списком серверов, на которых работаем. Может быть статическим (просто YAML или INI с IP-адресами) или динамическим (Ansible сам ходит в облако и получает актуальный список ВМ).
2. **Playbook** — YAML-файл с инструкциями: что и в каком порядке делать на каких серверах.
3. **Task** — одна задача (например, «установить nginx»).
4. **Module** — программа, которая выполняет конкретное действие (`apt` для пакетов, `copy` для файлов, `service` для systemd-сервисов, `lineinfile` для редактирования файлов).
5. **Role** — переиспользуемый набор задач, файлов, переменных. Например, роль `nginx` содержит всё, что нужно для установки и базовой настройки nginx.
6. **Handler** — задача, которая запускается только если её «уведомили» (например, перезапуск nginx после изменения конфига).
7. **Variables** — переменные, как в Terraform, чтобы один playbook работал для разных окружений.
8. **Galaxy** — публичный репозиторий готовых ролей: https://galaxy.ansible.com

> ℹ️ В 2026 году актуальная версия — **ansible-core 2.18+** (под капотом) и **Ansible 11+** (community package, объединяющий core + сотни коллекций модулей). Стабильно работает на Python 3.11+.

#### Пример простого playbook

```yaml
# install-web.yml
- name: Установка и настройка nginx
  hosts: web_servers
  become: true  # выполнять команды через sudo

  vars:
    nginx_port: 80
    welcome_message: "Сервер развёрнут через Ansible"

  tasks:
    - name: Обновить кэш apt и установить nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Развернуть индексную страницу из шаблона
      ansible.builtin.template:
        src: index.html.j2
        dest: /var/www/html/index.html
        owner: www-data
        group: www-data
        mode: "0644"
      notify: restart nginx  # если файл изменился — перезапустить nginx

    - name: Разрешить порт в ufw
      community.general.ufw:
        rule: allow
        port: "{{ nginx_port }}"
        proto: tcp

    - name: Убедиться, что nginx запущен и в автозагрузке
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

Файл шаблона `templates/index.html.j2` (Jinja2):
```html
<!DOCTYPE html>
<html>
<head><title>Hello from Ansible</title></head>
<body>
  <h1>{{ welcome_message }}</h1>
  <p>Host: {{ ansible_hostname }}</p>
  <p>IP: {{ ansible_default_ipv4.address }}</p>
</body>
</html>
```

Запуск: `ansible-playbook -i inventory.ini install-web.yml`

#### Структура Ansible-роли

```
roles/
└── nginx/
    ├── defaults/main.yml   # самые базовые переменные (можно перекрыть)
    ├── vars/main.yml       # переменные роли (приоритет выше)
    ├── tasks/main.yml      # сами задачи
    ├── handlers/main.yml   # обработчики (перезапуск сервисов)
    ├── templates/          # Jinja2-шаблоны
    ├── files/              # файлы для копирования как есть
    └── meta/main.yml       # метаданные (зависимости, автор)
```

### Terraform vs Ansible — когда что использовать

| Задача                                       | Чем сделать      |
|----------------------------------------------|------------------|
| Создать ВМ в облаке                          | Terraform        |
| Создать VPC, подсеть, балансировщик          | Terraform        |
| Создать Managed PostgreSQL                   | Terraform        |
| Установить nginx на существующую ВМ          | Ansible          |
| Положить конфиг и перезапустить сервис       | Ansible          |
| Обновить версию ОС на 100 серверах           | Ansible          |
| Создать Kubernetes-кластер в облаке          | Terraform        |
| Установить Helm-чарт в Kubernetes            | Helm (или Terraform с helm-провайдером) |
| Создать пользователей и роли в облаке (IAM)  | Terraform        |
| Подкрутить параметры ядра Linux на сервере   | Ansible          |

**Главное правило:** Terraform — про *создание и удаление* ресурсов в облаке. Ansible — про *состояние* программ и конфигов внутри уже существующих машин.

### Альтернативы и тренды 2026 года

1. **OpenTofu** — открытый форк Terraform под Linux Foundation. Полностью совместим с Terraform, но без угрозы изменения лицензии.
2. **Pulumi** — IaC на «настоящих» языках программирования (TypeScript, Python, Go, C#). Удобнее для разработчиков, но Terraform/OpenTofu по-прежнему стандарт в DevOps.
3. **Crossplane** — управление облачной инфраструктурой через Kubernetes-манифесты. Создаёшь YAML с типом `Database`, Crossplane создаёт реальную базу в Yandex Cloud или AWS.
4. **Ansible Automation Platform** (бывший Ansible Tower / AWX) — корпоративная UI-надстройка для запуска playbooks, расписаний, отчётности, RBAC.
5. **Atlantis** и **env0** — инструменты для GitOps-подхода к Terraform: pull request → автоматический plan в комментарии → ручное подтверждение apply.

### Теоретические вопросы

1. **Что такое Infrastructure as Code и в чём его преимущества?**
   **Ответ:** Подход, при котором вся инфраструктура описана в текстовых файлах, версионируется в Git и применяется автоматизированно. Преимущества: воспроизводимость, ревью изменений, версионирование, скорость разворачивания, отказ от ручных кликов, идентичные dev/staging/prod.

2. **В чём разница между декларативным и императивным подходом?**
   **Ответ:** Декларативный — описываешь *желаемый* результат (Terraform: «должно быть 3 ВМ»). Императивный — описываешь *шаги* для его достижения (bash: «создай ВМ, потом ещё одну»). Декларативный подход проще, безопаснее и легче поддаётся повторному применению.

3. **Что такое идемпотентность и почему она критична для IaC?**
   **Ответ:** Свойство операции: повторное выполнение даёт тот же результат, что и одно выполнение. Если запустить `terraform apply` или `ansible-playbook` дважды подряд, ничего не сломается и не создастся дважды.

4. **Что такое Terraform state и почему его нельзя терять?**
   **Ответ:** Файл `terraform.tfstate` — это «память» Terraform о том, какие ресурсы он создал и как они выглядят. Без state Terraform не знает, что управлять, и при следующем apply попытается создать всё заново или поломает существующее. Хранится в Object Storage с включённым versioning и блокировками.

5. **Что такое remote backend и блокировка state?**
   **Ответ:** Remote backend — хранение state не на ноутбуке инженера, а в общем хранилище (S3, Yandex Object Storage). Блокировка (locking) — механизм, при котором, пока один инженер запускает apply, другие не могут запустить параллельно — иначе state поломается.

6. **Чем `terraform plan` отличается от `terraform apply`?**
   **Ответ:** Plan показывает diff — что *будет* создано, изменено или удалено, но ничего не делает. Apply реально применяет изменения. Правило: всегда смотри plan перед apply, особенно на проде.

7. **Что такое модуль в Terraform?**
   **Ответ:** Переиспользуемый набор ресурсов с входными и выходными параметрами. Например, модуль «standard-vm» создаёт ВМ + диск + IP + DNS-запись. В разных проектах модуль вызывается с разными параметрами.

8. **Что такое provider в Terraform?**
   **Ответ:** Подключаемый модуль для работы с конкретным API: `yandex`, `aws`, `kubernetes`, `helm`, `postgresql`. Каждый provider добавляет свои ресурсы. В Terraform можно использовать несколько провайдеров одновременно.

9. **Можно ли запустить Terraform параллельно для одного проекта на двух машинах?**
   **Ответ:** Нет, если только не настроена блокировка state. Параллельный запуск без локов приведёт к рассинхронизации state и может «потерять» ресурсы из учёта.

10. **Что такое Ansible playbook?**
    **Ответ:** YAML-файл с описанием задач, которые Ansible выполнит на указанных в inventory машинах. Playbook состоит из *play* (групп задач для разных хостов) и *tasks* (отдельных операций).

11. **Что такое inventory и какие типы бывают?**
    **Ответ:** Список управляемых хостов. Статический — обычный INI/YAML с IP/именами. Динамический — скрипт или плагин, который при каждом запуске ходит в облако (Yandex Cloud, AWS) и получает актуальный список ВМ. В крупных инфраструктурах всегда используют динамический inventory.

12. **Что такое handler в Ansible и зачем он нужен?**
    **Ответ:** Задача, которая выполняется только если её уведомили (`notify`). Классический пример: после изменения конфига nginx нужно его перезапустить. Без handler перезапуск выполнялся бы каждый раз, даже если конфиг не менялся.

13. **Что такое идемпотентный модуль и приведи пример НЕидемпотентного использования.**
    **Ответ:** Модуль, который проверяет состояние перед действием. `ansible.builtin.apt: name=nginx state=present` идемпотентен — если nginx стоит, ничего не делает. Неидемпотентно: `command: apt install nginx` или `shell: echo "x" >> /etc/file` — будет добавлять строку в файл при каждом запуске. Для shell/command используют `creates:` или `removes:` для идемпотентности.

14. **Чем Ansible Role отличается от обычного playbook?**
    **Ответ:** Role — стандартизированная структура папок (tasks/, files/, templates/, vars/, handlers/, defaults/, meta/), которую можно переиспользовать в разных проектах.

15. **Чем Terraform отличается от Ansible и когда что использовать?**
    **Ответ:** Terraform — для *создания* ресурсов в облаке (ВМ, сети, БД). Ansible — для *настройки* программ и конфигов на уже существующих машинах. Terraform работает через API облака, Ansible — по SSH.

16. **Можно ли заменить Ansible на cloud-init / user_data?**
    **Ответ:** Для простой первоначальной настройки (установить 1–2 пакета, положить ключи) — да, cloud-init вполне работает и проще. Для сложной настройки, регулярных обновлений конфигов, многосерверных операций — нет, Ansible незаменим. Часто их комбинируют.

17. **Что такое Ansible Vault и зачем он нужен?**
    **Ответ:** Встроенный механизм шифрования секретов в YAML-файлах. Пароли, ключи, токены хранятся в зашифрованном виде в Git, расшифровываются при запуске playbook паролем или ключом. Не путать с HashiCorp Vault (он будет в уроке 9).

18. **Что такое OpenTofu и почему он появился?**
    **Ответ:** Открытый форк Terraform под управлением Linux Foundation. Появился после смены лицензии Terraform на BSL в августе 2023. По синтаксису и поведению полностью совместим с Terraform.

19. **Как проверить, что Terraform-код корректен, без реального применения?**
    **Ответ:** `terraform fmt` (формат), `terraform validate` (синтаксис и ссылки), `terraform plan` (что произойдёт), плюс инструменты `tflint`, `tfsec`, `checkov`, `trivy` для статического анализа на безопасность.

20. **Что такое drift и как его обнаружить?**
    **Ответ:** Drift — это когда реальное состояние инфраструктуры разошлось с тем, что описано в Terraform-коде (кто-то изменил ресурс руками через консоль). Обнаруживается через `terraform plan` — он покажет разницу. На проде стоит регулярно запускать `plan` по расписанию (через CI или Atlantis) и оповещать при drift.

### Вопросы с собеседований

1. **Каким инструментом ты бы создавал VPC, виртуальные машины и выделенные IP-адреса?**
   **Ответ:** Terraform или OpenTofu. Это создание ресурсов в облаке, где нужен учёт того, что уже создано, — за него отвечает state-файл. Ansible теоретически тоже умеет обращаться к API облаков, но не хранит состояние и потому не может корректно удалять и изменять ранее созданное

2. **А каким — установку Docker на машину, обновление сервиса и выдачу прав на каталог?**
   **Ответ:** Ansible. Это настройка уже существующей машины по SSH: пакеты, файлы, права, перезапуск сервисов. Terraform сюда не подходит — он оперирует ресурсами облака, а не содержимым операционной системы

3. **Как выглядит структура Ansible-роли?**
   **Ответ:** Каталог роли содержит `tasks/main.yml` (задачи), `handlers/main.yml` (обработчики, запускаемые по `notify`), `templates/` (шаблоны Jinja2), `files/` (файлы для копирования как есть), `vars/main.yml` (переменные с высоким приоритетом), `defaults/main.yml` (значения по умолчанию, самый низкий приоритет), `meta/main.yml` (зависимости)

4. **Чем `vars` отличается от `defaults`?**
   **Ответ:** Приоритетом. `defaults` — значения по умолчанию, которые пользователь роли легко переопределит. `vars` перебивает почти всё и меняется трудно. Практическое правило: всё, что пользователь роли может захотеть изменить, кладут в `defaults`

5. **Что такое handler и чем он лучше обычной задачи?**
   **Ответ:** Задача, которая выполняется только если её вызвали через `notify`, и только один раз в конце запуска, сколько бы раз её ни вызвали. Классический пример — перезапуск nginx: конфиг могли поменять три задачи, но перезапустить сервис нужно однократно и только если что-то действительно изменилось

6. **Где Terraform хранит информацию о созданном и почему это важно?**
   **Ответ:** В state-файле. Именно он позволяет понять, что уже существует, что нужно изменить и что удалить. Хранить его локально нельзя: коллега не увидит твоих изменений, а одновременный запуск двух `apply` разрушит состояние. Правильно — удалённое хранилище с версионированием и блокировкой

7. **Что произойдёт, если удалить state-файл?**
   **Ответ:** Terraform «забудет» про созданные ресурсы: в облаке они останутся и продолжат тратить деньги, а следующий `apply` попытается создать их заново и, скорее всего, упадёт на конфликте имён. Восстанавливают либо из версионированного хранилища, либо командой `terraform import` по каждому ресурсу вручную — это долго и больно

### Практическая часть

#### Задание 1. Установка инструментов

**Шаги:**

1. Установи Terraform или OpenTofu. В России лучше OpenTofu (нет лицензионных ограничений):
   ```bash
   curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh \
     -o install-opentofu.sh
   chmod +x install-opentofu.sh
   ./install-opentofu.sh --install-method standalone
   tofu --version
   ```

2. Установи Ansible:
   ```bash
   python3 -m pip install --user ansible --break-system-packages
   ansible --version
   # Должно быть ansible-core 2.18+
   ```

3. Проверь, что у тебя настроен `yc` CLI и SSH-ключ из урока 6.

#### Задание 2. Первая инфраструктура через Terraform

**Цель:** Создать ВМ в Yandex Cloud полностью декларативно.

**Шаги:**

1. Создай папку и базовые файлы:
   ```bash
   mkdir terraform-lesson && cd terraform-lesson
   touch main.tf .gitignore
   ```

2. Содержимое `.gitignore` (state-файлы не должны попадать в Git!):
   ```
   .terraform/
   *.tfstate
   *.tfstate.backup
   *.tfvars
   crash.log
   ```

3. Заполни `main.tf` примером кода из теоретической части (тот, что выше).

4. Инициализация и применение:
   ```bash
   tofu init           # или terraform init
   tofu fmt
   tofu validate
   tofu plan           # внимательно прочитай вывод
   tofu apply          # подтверди yes
   ```

5. Посмотри `terraform.tfstate` — это и есть «память» Terraform.

6. Сделай изменение (например, поменяй имя ВМ) и снова `plan` → `apply`.

7. **Удали всё:**
   ```bash
   tofu destroy
   ```

#### Задание 3. State в Object Storage

**Цель:** Перенести state в remote backend.

**Шаги:**

1. Создай bucket для state через `yc storage bucket create`.

2. Создай сервисный аккаунт со static access key:
   ```bash
   yc iam service-account create --name tf-state
   yc iam access-key create --service-account-name tf-state
   ```

3. Добавь блок `backend "s3"` в `main.tf` (как в примере выше).

4. Передай ключи через переменные окружения:
   ```bash
   export AWS_ACCESS_KEY_ID="<твой access key>"
   export AWS_SECRET_ACCESS_KEY="<твой secret>"
   tofu init -migrate-state
   ```

5. Проверь в Object Storage, что файл state появился.

#### Задание 4. Первый Ansible playbook

**Цель:** Настроить ВМ, созданную Terraform, через Ansible.

**Шаги:**

1. Создай inventory.ini:
   ```ini
   [web]
   <публичный_IP_твоей_ВМ> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
   ```

2. Проверь связь:
   ```bash
   ansible -i inventory.ini web -m ping
   # должно вернуть pong
   ```

3. Создай playbook `web.yml`:
   ```yaml
   - name: Базовая настройка веб-сервера
     hosts: web
     become: true
     tasks:
       - name: Обновить apt cache
         ansible.builtin.apt:
           update_cache: true
           cache_valid_time: 3600

       - name: Установить пакеты
         ansible.builtin.apt:
           name:
             - nginx
             - htop
             - curl
             - jq
           state: present

       - name: Развернуть индексную страницу
         ansible.builtin.copy:
           dest: /var/www/html/index.html
           content: |
             <html><body>
             <h1>Привет от Ansible!</h1>
             <p>Хостнейм: {{ ansible_hostname }}</p>
             <p>Дата: {{ ansible_date_time.date }}</p>
             </body></html>
           owner: www-data
           group: www-data
           mode: "0644"

       - name: Запустить и включить nginx
         ansible.builtin.service:
           name: nginx
           state: started
           enabled: true
   ```

4. Запусти:
   ```bash
   ansible-playbook -i inventory.ini web.yml
   ```

5. Проверь в браузере: `http://<IP_ВМ>` — должна открыться твоя страница.

6. Запусти playbook ещё раз. Обрати внимание, что Ansible выдаёт `ok` вместо `changed` — это работа идемпотентности.

7. Измени текст и запусти ещё раз. На этот раз будет `changed=1` — Ansible увидел разницу.

#### Задание 5. Свой Ansible role

**Цель:** Превратить плоский playbook в роль.

**Шаги:**

1. Сгенерируй структуру роли:
   ```bash
   ansible-galaxy init roles/nginx
   ```

2. Перенеси задачи из `web.yml` в `roles/nginx/tasks/main.yml`, шаблон страницы — в `roles/nginx/templates/index.html.j2`.

3. Используй модуль `ansible.builtin.template` вместо `copy`, чтобы поддерживать Jinja2-переменные.

4. В `roles/nginx/defaults/main.yml` положи дефолтные переменные:
   ```yaml
   nginx_welcome_title: "Привет от роли"
   ```

5. Обнови основной playbook:
   ```yaml
   - hosts: web
     become: true
     roles:
       - nginx
   ```

#### Задание 6. Связка Terraform + Ansible

**Цель:** Получить полный pipeline «создать ВМ → настроить ВМ».

**Шаги:**

1. После `tofu apply` Terraform выведет публичный IP в `output`.

2. Создай скрипт `apply-all.sh`:
   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   tofu apply -auto-approve

   IP=$(tofu output -raw vm_public_ip)
   echo "[*] Жду 30 секунд, пока ВМ полностью загрузится..."
   sleep 30

   # Динамический inventory
   echo "[web]" > inventory.ini
   echo "$IP ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_ssh_common_args='-o StrictHostKeyChecking=no'" >> inventory.ini

   ansible-playbook -i inventory.ini web.yml
   ```

3. Запускай `bash apply-all.sh` — и у тебя будет полностью готовый веб-сервер за пару минут, без единого клика мышью.

4. **Удали всё:** `tofu destroy`

#### Задание 7. Сквозной проект: инфраструктура «Заметок» как код

**Цель:** воспроизвести результат темы 6 (приложение на сервере в интернете) — но теперь целиком кодом, без единого клика, и с возможностью повторить это в любом окружении

**Шаги:**

1. Создай каталог `infra/` в репозитории «Заметок» и опиши в `main.tf` сеть, группу безопасности и машину. Часть кода бери из примера в начале темы; ключевое отличие — правила доступа:

   ```hcl
   resource "yandex_vpc_security_group" "notes" {
     name       = "notes-sg"
     network_id = yandex_vpc_network.main.id

     # Правило для SSH: только с твоего адреса, а не со всего интернета
     ingress {
       protocol       = "TCP"
       port           = 22
       v4_cidr_blocks = [var.my_ip_cidr]
     }

     ingress {
       protocol       = "TCP"
       port           = 80
       v4_cidr_blocks = ["0.0.0.0/0"]
     }

     # Наружу можно всё: серверу нужно скачивать пакеты и образы
     egress {
       protocol       = "ANY"
       from_port      = 0
       to_port        = 65535
       v4_cidr_blocks = ["0.0.0.0/0"]
     }
   }
   ```

2. Добавь `outputs.tf`, чтобы Terraform сам сообщил адрес созданной машины:

   ```hcl
   output "notes_ip" {
     description = "Публичный адрес сервиса «Заметки»"
     value       = yandex_compute_instance.web[0].network_interface.0.nat_ip_address
   }
   ```

3. Опиши настройку машины в `ansible/notes.yml` — то, что в теме 6 ты делал руками:

   ```yaml
   - name: Развернуть сервис «Заметки»
     hosts: notes
     become: true

     tasks:
       - name: Установить Docker и git
         ansible.builtin.apt:
           name: [docker.io, docker-compose-v2, git]
           state: present
           update_cache: true

       - name: Забрать код приложения
         ansible.builtin.git:
           repo: https://github.com/<твой_логин>/notes.git
           dest: /opt/notes
           version: main
         # register сохраняет результат задачи, чтобы ниже узнать, менялся ли код
         register: repo

       - name: Поднять стенд через Compose
         ansible.builtin.command:
           cmd: docker compose up -d --build
           chdir: /opt/notes
         # Перезапускаем только если код действительно обновился — это и есть идемпотентность
         when: repo.changed

       - name: Дождаться, пока приложение начнёт отвечать
         ansible.builtin.uri:
           url: http://localhost:8080/healthz
           status_code: 200
         register: health
         until: health.status == 200
         retries: 12
         delay: 5
   ```

4. Собери всё в один сценарий `deploy.sh`:

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail

   cd infra
   tofu init
   tofu apply -auto-approve
   IP=$(tofu output -raw notes_ip)
   cd ..

   echo "[*] Машина создана: ${IP}"
   echo "[*] Жду загрузки системы"
   sleep 30

   printf '[notes]\n%s ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519 ansible_ssh_common_args="-o StrictHostKeyChecking=no"\n' \
     "${IP}" > ansible/inventory.ini

   ansible-playbook -i ansible/inventory.ini ansible/notes.yml
   echo "[*] Готово: http://${IP}/"
   ```

5. Запусти, проверь, **запусти ещё раз**:

   ```bash
   bash deploy.sh
   curl "http://$(cd infra && tofu output -raw notes_ip)/"
   bash deploy.sh          # второй запуск: ничего не пересоздаётся
   ```

   Второй запуск — главная проверка темы. Terraform скажет `No changes`, Ansible покажет `ok` вместо `changed` почти во всех задачах. Это идемпотентность: повторное применение приводит систему в то же состояние, а не ломает её

6. Снеси всё и разверни заново:

   ```bash
   cd infra && tofu destroy -auto-approve && cd ..
   bash deploy.sh
   ```

   Полный цикл «нет ничего → работающий сервис» занял пару минут и ноль кликов. Ради этого и существует IaC

7. Закоммить:

   ```bash
   git add infra/ ansible/ deploy.sh
   git commit -m "Описать инфраструктуру «Заметок» кодом"
   git push
   ```

**Ожидаемый вывод:** `tofu output` печатает адрес; сайт открывается; повторный запуск ничего не меняет; после `destroy` в облаке не остаётся ресурсов

**Типичные ошибки:**
- `terraform.tfstate` попал в git — там могут быть пароли и внутренние адреса. Добавь в `.gitignore` строки `*.tfstate`, `*.tfstate.*`, `.terraform/`, а сам state держи в Object Storage, как в задании 3
- Ansible не подключается сразу после `apply` — машина ещё загружается. Отсюда `sleep` в сценарии; более правильное решение — модуль `wait_for_connection`
- Задача с `command` всегда показывает `changed` — так и есть, Ansible не знает, что делает произвольная команда. Поэтому в примере стоит `when: repo.changed`, а по возможности используют специализированные модули вместо `command`
- `destroy` не удаляет то, что создано руками в консоли. Terraform управляет только тем, что есть в его state — смешивать ручное и кодовое управление одним ресурсом нельзя

#### Задание 8. Чек-лист зрелого использования IaC

Проверь себя, можешь ли ты ответить «да» на эти пункты:

- [ ] Весь Terraform-код лежит в Git
- [ ] State хранится не локально, а в Object Storage с включённым versioning
- [ ] Используется блокировка state при `apply`
- [ ] Перед `apply` всегда смотрится `plan`
- [ ] Изменения в проде идут через Pull Request с code review
- [ ] CI проверяет код: `terraform fmt -check`, `terraform validate`, `tflint`, `tfsec`
- [ ] Секретные данные (пароли, ключи) не лежат в Git в открытом виде
- [ ] Ansible-роли версионируются и переиспользуются между проектами
- [ ] Для секретов в Ansible используется `ansible-vault`
- [ ] Есть тесты Ansible-ролей через Molecule

Если на все пункты «да» — у тебя зрелая IaC-практика.

### Проверь себя: тема освоена, если ты можешь

- [ ] Объяснить, что такое идемпотентность, и показать её на своём коде
- [ ] Объяснить разницу между Terraform и Ansible и сказать, что чем делать
- [ ] Рассказать, что хранится в state-файле и почему его нельзя держать в git
- [ ] Объяснить, почему `plan` смотрят перед каждым `apply`
- [ ] Написать playbook, который не выполняет лишней работы при повторном запуске
- [ ] Развернуть окружение с нуля и полностью удалить его одной командой
- [ ] Объяснить, почему изменения инфраструктуры проводят через Pull Request

### [Следующая тема: метрики, логирование, трейсинг →](8-Метрики-логгирование-трейсинг-istio.md)
