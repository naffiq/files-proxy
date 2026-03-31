# files-proxy

Nginx reverse-proxy для доступа к файлам S3-хранилищ PS Cloud через единый домен `files.emdey.kz`. Решает проблему CORS при работе с файлами из фронтенда.

## Проблема

S3-хранилища (`object.pscloud.io`, `archive.pscloud.io`) не возвращают нужные CORS-заголовки, из-за чего браузер блокирует запросы к файлам с фронтенда (`mis.emdey.kz`).

## Как это работает

Прокси принимает запросы на `files.emdey.kz` и перенаправляет их на соответствующее S3-хранилище, добавляя CORS-заголовки в ответ. Все query-параметры (включая pre-signed URL подписи) пробрасываются без изменений.

## Подмена URL

### Обычное хранилище (object)

```
# Было (S3 напрямую):
https://<bucket>.object.pscloud.io/<file-path>?<query-params>

# Стало (через прокси):
https://files.emdey.kz/object/<bucket>/<file-path>?<query-params>
```

### Архивное хранилище (archive)

```
# Было (S3 напрямую):
https://<bucket>.archive.pscloud.io/<file-path>?<query-params>

# Стало (через прокси):
https://files.emdey.kz/archive/<bucket>/<file-path>?<query-params>
```

### Примеры

| S3 URL | Proxy URL |
|--------|-----------|
| `https://prod-service-results.object.pscloud.io/report.pdf` | `https://files.emdey.kz/object/prod-service-results/report.pdf` |
| `https://prod-service-results.archive.pscloud.io/old-report.pdf` | `https://files.emdey.kz/archive/prod-service-results/old-report.pdf` |
| `https://dev-holding-logo.object.pscloud.io/logo.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&...` | `https://files.emdey.kz/object/dev-holding-logo/logo.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&...` |

### Подмена в коде

```js
function toProxyUrl(s3Url) {
  const url = new URL(s3Url);
  const [bucket, storage] = url.hostname.split('.');
  // storage = "object" или "archive"
  return `https://files.emdey.kz/${storage}/${bucket}${url.pathname}${url.search}`;
}
```

## Запуск

```bash
docker build -t files-proxy .
docker run -p 80:80 files-proxy
```
