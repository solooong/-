# Если что-то не работает

## `docker-compose build` выдаёт что-то про `no space left on device`

Радикальное _(но не умное)_ решение, которое точно сработает: удалить все volumes, весь билд-кэш, все образы и все слои.

```bash
docker system prune -af --volumes
```

## Под Windows Next в докере не реагирует на изменение файлов

Добавить в `next.config.js`:

```js
  webpackDevMiddleware: (config) => {
    config.watchOptions = {
      poll: 1000, // Check for changes every second
      aggregateTimeout: 300, // delay before rebuilding
    };
    return config;
  },
```