# Настройка окружения

_Настройка требуется только для нормальной работы IDE, а код приложения выполняться будет в docker_

- Установить [nvm](https://github.com/nvm-sh/nvm) (Node Version Manager) , далее выполнять команды в директории webapp:

```bash
# Выбрать требуемую версию node.js из файла .nvmrc
# (используется та же, что и в Dockerfile)
nvm use

# установить нужную версию node.js
# (выполнится только если она ещё не установлена)
nvm install

# Установить зависимости проекта (автоматически выполнится graphql-codegen)
npm install

```

## Настройка редактора

### Другие редакторы

- Настроить так, чтобы форматтером работал локальный Eslint с его локальной конфигурацией _(локальный = установленный в проекте)_
- Добавить поддержку SCSS

### VSCode

- Для нормальной работы расширения `eslint` лучше открывать в vscode не директорию `platform` , а директорию с веб-приложением.
- Установить расширение `dbaeumer.vscode-eslint` , включить в нём режим форматтера (отдельнно), назначить его форматтером по умолчанию для js и ts, а затем перезагрузить VSCode (полностью выйти):

```json
{
  "eslint.format.enable": true,
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "[javascript]": {
    "editor.defaultFormatter": "dbaeumer.vscode-eslint",
    "editor.formatOnSave": true
  },
  "[javascriptreact]": {
    "editor.defaultFormatter": "dbaeumer.vscode-eslint",
    "editor.formatOnSave": true
  },
  "[typescript]": {
    "editor.defaultFormatter": "dbaeumer.vscode-eslint",
    "editor.formatOnSave": true
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "dbaeumer.vscode-eslint",
    "editor.formatOnSave": true
  }
}
```

- Затем нужно открыть любой TS-файл и ввести команду `TypeScript: Velect TypeScript Version` > `Use Workspace version` . Теперь VSCode будет использовать ту же самую версию TS, что и next.
- Для поддержки IntelliSense в SCSS - `mrmlnc.vscode-scss`
- Для валидации SCSS - `stylelint.vscode-stylelint`.
- Для подсветки запросов и схемы GraphQL нужно установить расширение `prisma.vscode-graphql`, а с форматированием GraphQL справляется `esbenp.prettier-vscode`.
- Для подсветки снепшотов Jest - расширение `tlent.jest-snapshot-language-support`

## Настройка браузера

### Другие браузеры

- Установить расширение React Developer Tools.
- Установить расширение, отключающее CORS (наш бэкенд не умеет дописывать заголовки для localhost при работе с dev-окружением).

### Chrome

- Установить расширение [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) для Google Chrome.
- Установииь расширение [Allow CORS: Access-Control-Allow-Origin](https://chrome.google.com/webstore/detail/allow-cors-access-control/lhobafahddgcelffkeicbaginigeejlf) для того, чтобы отключить CORS (наш бэкенд не умеет дописывать заголовки для localhost при работе с dev-окружением).
