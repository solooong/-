# Webapp-tayga

- [C чем должен быть знаком разработчик для работы над этим кодом](Документация/11.%20webapp-taiga/docs/onboarding.md)
- [Code style](Документация/11.%20webapp-taiga/docs/code-style.md)
- [Настройка окружения и редактора](Документация/11.%20webapp-taiga/docs/editor-setup.md)
- [Загрузка картинок на сервер](Документация/11.%20webapp-taiga/docs/images-upload.md)
- [Структура проекта](Документация/11.%20webapp-taiga/docs/structure.md)
- [Принятые решения](Документация/11.%20webapp-taiga/docs/decisions.md)
- [Техдолг, костыли, нерешённые вопросы](./docs/todo.md)
- [Комментарии к зависимостям](./package.json)
- [Если что-то не работает](Документация/11.%20webapp-taiga/docs/troubleshooting.md)
- [Отчёт о покрытии тестами (появляется после запуска тестов, смотрится в браузере)](coverage/lcov-report/index.html)

## Порядок разработки и отладки

- Для запуска вебаппа в процессе разработки прежде всего нужно раскомментировать соответствующие ему поля в `docker-compose.override.yml` и выполнить `docker-compose build webapp-taiga`.
- Когда изменились зависимости (package.json) или graphql-запросы,
  надёжнее всего будет заново собрать образ вебаппа и перезапустить сервис, не сохраняя volume
  `docker-compose up -V --build webapp-taiga`, а затем ещё выполнить локально `npm install` (при этом `codegen` выполнится автоматически), чтобы интеграция с редактором работала как следует.
- Логи в консоли браузера будут видны только для тех действий, которые выполняются после SSR (даже в dev-режиме!). Логи SSR нужно смотреть в консоли: `docker-compose logs -f --tail=200 webapp-taiga`

## Команды npm

Команды, приведённые ниже, выполняются в корневой директории `platform` . Выполнять их можно:

- локально
- через `docker-compose run` (запустить контейнер с другим entrypoint)
- в уже запущенном контейнере через `docker-compose exec`

```bash
# Запустить тесты и следить за изменениями файлов
npm test

# Запустить тесты единовременно, без отчёта о покрытии
npm run test-ci

# Проверить с eslint, stylelint
npm run check

# Иcправить замечения eslint (для случаев, когда редактор не интегрирован с eslint)
npm run eslint-fix

# Сгенерить типы TypeScript из схемы graphql
npm run codegen

# Open Storybook (component library) at http://localhost:6006 
npm run storybook

# Запуск в докере из директории webapp-taiga
docker-compose -f ../docker-compose.yml -f ../docker-compose.override.yml -f ../docker-compose.frontend-dev.yml up -d webapp-taiga
```
