                                 Всем привет !


  При написании был частично использован материал отсюда https://github.com/skl256/grafana_stack_for_docker.git ,
а также отсюда https://www.dmosk.ru/miniinstruktions.php?mini=prometheus-stack-docker . Но последний источник
сильно устарел.
  Большое количество rules можно посмотреть на https://samber.github.io/awesome-prometheus-alerts/rules .

  Скачиваем с GitHub-а :
  git clone https://github.com/dimonpoc/metrics-telegram-1.git

  Вносим изменения в alertmenager.yml (см. ниже).

  Запускаем Docker :
  docker compose up -d


### Prometheus

Документация: <https://prometheus.io/docs/alerting/latest/configuration/>

* Образ `prom/prometheus:latest`
* Порт `9090`
* Команда: `command: ["--config.file=/etc/prometheus/prometheus.yaml", "--web.config.file=/etc/prometheus/web-config.yaml", "--web.enable-lifecycle"]` (`web.enable-lifecycle` для перечитывания `web-config` "на лету")

Пример `web-config.yaml` (используется для basic_auth)
<https://github.com/prometheus/prometheus/blob/main/documentation/examples/web-config.yaml>

* password_hash генерируется командой `htpasswd -nBC 10 "" | tr -d ':\n'` 
  * Если htpasswd не установлен: для Debian, Ubuntu `sudo apt install apache2-utils`
  * для ОС использующих yum `sudo yum install -y httpd-tools`

```
basic_auth_users:
  admin: $2y$10$K7gXeAs0VbhjHMdlV1Hn0OlWcqIoK7P9s/dVKB3HoyYcLuscxSpXe # change "$2y$10..." to basic auth password_hash
  bobi: $2y$10$1sYkKxi49lGpFdlu7aDkTeWkzvkqaeCTb4PDBR/pNxeETO8N3shZS # you can add more users
```
* В нашем случае аутентификация отключена , при желании это надо делать самостоятельно .

* Время и максимальный объём хранимых логов можно настроить с помощь команд запуска `--storage.tsdb.retention.time=15d`, `--storage.tsdb.retention.size=0` (напр. `512MB`) 
  * подробнее: <https://prometheus.io/docs/prometheus/latest/storage/>

  В данном примере alert-ы в alert.rules используем просто как поставщики статистики , но не для 
вычисления неких граничных условий .
  Так же в качестве поставщика используем только Node Exporter , хотя вариантов масса .

### Node-Exporter

  Собираем метрики с железа .

### Alertmanager

  Посылаем сообщения на Telegram .
  Необходимо в Telegram-е создать бот .
  В alertmanager.yml в блоке receivers указать идентификационный номер Telegram ID ( chat_id ) без апостофов , и 
указать токен бота ( bot_token ) с апострофами . Токен бота является конфиденциальной информацией .
