# Организация процесса разработки

Здесь описаны действия, необходимые для начала разработки, а также различные вспомогательные инструменты которые
используются в процессе разработки

### Подготовка окружения

Для разработки требуется установленные:

- `python 3.9`
- `libpq-dev` и `python3.9-dev` для корректной работы `psycopg2`
  ```bash 
    sudo apt-get install libpq-dev python3.9-dev
    ```

- все зависимости, указанные в `server-realty/requirements.txt`
- [`docker`](https://docs.docker.com/get-docker/) и [`docker-compose`](https://docs.docker.com/compose/install/)

### Запуск приложения и частые операции

Часто используемые варианты запуска приложения и вспомогательные действия реализованы в приведённых ниже скриптах,
которые удобно использовать как команды-псевдонимы (aliases).

##### Запуск сервера

	* `realty-platform-server-localhost` - поднять сервер локально + зависимости сервера в контейнере
    * для разработки обычно используется именно этот вариант запуска
    * при запуске в данном варианте работает live reload
* `realty-platform-server-container` - поднять сервер в контейнере + зависимости сервера в контейнере
    * используется редко
* `realty-platform-full-stack` - поднять сервер в контейнере + зависимости сервера в контейнере + фронтенд в контейнере
    * полезен для тестирования взаимодействия фронтенда с сервером
    * используется редко
* `realty-platform-server-localhost-restart` - перезапустить локальный сервер с актуализацией данных схемы graphql
    * полезен для локальной разработки с частыми изменениями в схеме graphql
    * не перезапускает зависимости в контейнерах

Также иногда бывает полезно запустить фронтенд в контейнере и заставить его использовать бэкенд-сервер, поднятый
локально вне контейнера. В таком варианте можно пользоваться преимуществами live reload и при этом видеть как изменения
в сервере влияют на поведение фронтенда. Для этого необходимо сделать следующее:

* поднять бэкенд локально (алиас `realty-platform-server-localhost`)
* прописать в `docker-compose.yml` для сервиса `webapp-taiga` значение переменной
  среды `GRAPHQL_SSR_URL: "http://localhost:8000/"`
* также прописать для сервиса `webapp-taiga` параметр `network_mode: host`
* поднять фронтенд в контейнере командой `docker-compose up webapp-taiga`

##### Работа с тестами

* `realty-platform-tests-localhost-setup` - поднять зависимости тестов в контейнере
    * обычно необходимо выполнить эту команду единожды перед началом работы с тестами
* `realty-platform-tests-localhost` - запустить тесты локально
    * для работы с тестами обычно используется именно эта команда
    * параметр `l` выводит логи в stdout
    * параметр `t` позволяет запустить один или несколько конкретных тестов / классов тестов / модулей тестов
* `realty-platform-tests-container` - поднять зависимости тестов в контейнере, затем прогнать тесты в контейнере
    * в данный варианте работают тесты на CI, поэтому команда может быть полезна для диагностики проблем с тестами
      которые проявляются только на CI
    * В повседневной разработке практически не используется

Пример запуска индивидуального теста локально:

```bash
$ realty-platform-tests-localhost -lt tests.tests.core.queries.test_flat_offer_list.FlatOfferListQuerySortingTest.test_that_flat_offer_list_is_sorted_by_price_ascending   
```

##### Вспомогательные команды

* `realty-platform-voyager` - просмотр визуализации схемы graphql
* `realty-platform-makemigrations` - создание миграций c помощью django

#### Установка alises

Чтобы сохранить алиасы в системе необходимо:

* прописать в переменной `PROJECT_ROOT` путь к корневой папке проекта на вашей локальной машине
* скопировать код в `~/.bashrc`
* актуализировать алиасы - выполнить команду `source ~/.bashrc` или перелогиниться

```bash
# change this to your local project root, e.g. '~/repositories/realty-platform'
PROJECT_ROOT=<PATH_TO_PROJECT_ROOT>

SERVER_ROOT=$PROJECT_ROOT/server-realty/server_realty

# == aliases ==

# run server locally
alias realty-platform-server-localhost='(
	# copy graphql schema
	cd $PROJECT_ROOT
	find $SERVER_ROOT/graphql_api/schema -name "*.graphql" -type f -delete
	cp -r graphql-schema/* server-realty/server_realty/graphql_api/schema/
	
	# calculate graphql schema hash and save to file
	cd $SERVER_ROOT
	md5sum graphql_api/schema/*.graphql graphql_api/schema/**/*.graphql | cut -d " " -f 1 | md5sum | cut -d " " -f 1 > server_realty/schema-hash.txt
	
	# set local dev settings
	export DJANGO_SETTINGS_MODULE=server_realty.settings_dev
	
	# stop containerized server (if running)
	cd $PROJECT_ROOT
	docker-compose down --remove-orphans server-realty >/dev/null 2>&1
	
	# run containers with dependencies
	cd $PROJECT_ROOT
	docker-compose up --build --remove-orphans -dV server-realty-beat server-realty-worker flower
	
	# run server
	cd $SERVER_ROOT
        sh ./run_server.sh
)'

# restart local server (does not restart containerized dependencies)
alias realty-platform-server-localhost-restart='(
	# copy graphql schema
	cd $PROJECT_ROOT
	find $SERVER_ROOT/graphql_api/schema -name "*.graphql" -type f -delete
	cp -r graphql-schema/* server-realty/server_realty/graphql_api/schema/
	
	# calculate graphql schema hash and save to file
	cd $SERVER_ROOT
	md5sum graphql_api/schema/*.graphql graphql_api/schema/**/*.graphql | cut -d " " -f 1 | md5sum | cut -d " " -f 1 > server_realty/schema-hash.txt
	
	# set local dev settings
	export DJANGO_SETTINGS_MODULE=server_realty.settings_dev
	
	# run server
	cd $SERVER_ROOT
	sh ./run_server.sh
)'

# run server using container
alias realty-platform-server-container='(
    cd $PROJECT_ROOT
	docker-compose up --build --remove-orphans -dV server-realty
)'

# prep environment for running tests locally
alias realty-platform-tests-localhost-setup='(
	# copy graphql schema
	cd $PROJECT_ROOT
	find $SERVER_ROOT/graphql_api/schema -name "*.graphql" -type f -delete
	cp -r graphql-schema/* server-realty/server_realty/graphql_api/schema/

	# run containers with dependencies
	cd $PROJECT_ROOT
	docker-compose -f docker-compose.run-server-tests.yml up --build --remove-orphans -dV test-db test-local-redis-sentinel
)'

# run tests locally
alias realty-platform-tests-localhost='
	# copy graphql schema
	cd $PROJECT_ROOT
	find $SERVER_ROOT/graphql_api/schema -name "*.graphql" -type f -delete
	cp -r graphql-schema/* server-realty/server_realty/graphql_api/schema/

	# calculate graphql schema hash and save to file
	cd $SERVER_ROOT
	md5sum graphql_api/schema/*.graphql graphql_api/schema/**/*.graphql | cut -d " " -f 1 | md5sum | cut -d " " -f 1 > server_realty/schema-hash.txt

	# run tests (accepts arguments)
	cd $SERVER_ROOT
	python runtests.py'

# run tests using container
alias realty-platform-tests-container='(
	cd $PROJECT_ROOT 
	docker-compose -f docker-compose.run-server-tests.yml up --build --remove-orphans --exit-code-from test-server-realty test-server-realty
	docker-compose -f docker-compose.run-server-tests.yml down -v --remove-orphans
)'

# run server and frontend webapp using containers
alias realty-platform-full-stack='(
    cd $PROJECT_ROOT
	docker-compose up --build --remove-orphans -dV webapp-taiga server-realty flower
)'

# generate django database migrations
alias realty-platform-makemigrations='(
	cd $PROJECT_ROOT
	find $SERVER_ROOT/graphql_api/schema -name "*.graphql" -type f -delete
	cp -r graphql-schema/* server-realty/server_realty/graphql_api/schema/
	docker-compose up --build --remove-orphans -dV db vault-ssh-tunnel

	cd $SERVER_ROOT
	export DJANGO_SETTINGS_MODULE="server_realty.settings_dev"
	python manage.py makemigrations
)'

# run graphql-voyager using container
alias realty-platform-voyager='(
	cd $PROJECT_ROOT
	docker-compose up --build --remove-orphans -d graphql-voyager

	nohup xdg-open http://localhost:8080/ >/dev/null 2>&1
)'
```

Альтернативный вариант - последовательно выполнять каждую из команд в алиасе реализующем нужный вариант запуска

### Вспомогательные инструменты

* `pgadmin` - прямой доступ к содержимому Postgres
    * запуск: `docker-compose up pgadmin`
    * доступен по адресу [http://localhost:5555](http://localhost:5555)
* `redis insight` - прямой доступ к содержимому Redis
    * запуск: `docker-compose up redis-admin`
    * доступен по адресу [http://localhost:6050](http://localhost:6050)
* `graphql voyager` - визуализация схемы graphql
    * запуск: `docker-compose up --build graphql-voyager` либо вышеописанный alias `realty-platform-voyager`
    * доступен по адресу [http://localhost:8080](http://localhost:8080)
* визуализация отношений моделей django (т.е. схемы базы данных)
    * доступна при запуске сервера со флагом `DEBUG=True` по
      адресу [http://localhost:8000/models/](http://localhost:8000/models/)

### Внешние инструменты и сервисы

* [консоль Yandex.Cloud](https://console.cloud.yandex.ru/cloud?section=overview)
    * доступ к данным о состоянии виртуальных машин на окружениях
    * доступ к данным об использовании ресурсов на различных составляющих архитектуры
    * прямой доступ к содержимому object storage
    * доступ к базам на окружениях с возможностью выполнения SQL-запросов, а также с инструментами для визуализации
      результатов `EXPLAIN ANALYZE`

* [Logz.io](https://logz.io/) - хранилище недавних логов с возможностью анализа и визуализации
    * Kibana as a service
    * на данный момент имеет дэшборды с визуализацией данных про обработку фидов и изображений

* [Rollbar](https://rollbar.com/) - регистрация исключений

#### Непосредственный доступ к машинам в облаке

Инструкция по подключению к машинам в облаке расположена в [документации инфраструктуры](/docs/infrastructure.md).
