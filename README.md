# Certbot Cloudflare DNS Docker Container with webhook support

Этот контейнер используется для генерации и автоматического обновления SSL-сертификатов Let's Encrypt с помощью плагина Cloudflare DNS. Он основан на официальном образе [Certbot](https://hub.docker.com/r/certbot/dns-cloudflare) с некоторыми изменениями и дополнениями, делающими его более гибким и настраиваемым. Поддерживает deploy hook после обновления сертификата.


## Базовый образ

Образ основан на `certbot/dns-cloudflare:latest`, обеспечивая стабильную и актуальную среду для работы Certbot с аутентификацией Cloudflare DNS.


## Возможности
 
- Автоматическая генерация и обновление SSL-сертификата с помощью Let's Encrypt
- Конфигурации не требуются, этот образ автоматически генерирует файл `cloudflare.ini`.
- Аутентификация Cloudflare DNS для проверки домена
- Настраиваемая конфигурация с помощью переменных среды
- Периодические проверки обновления сертификатов
- Поддержка Windows (установка `REPLACE_SYMLINKS = true`)
- Встроенная проверка healthcheck 
- Скрипт, который запускается после успешного обновления SSL-сертификата


## Переменные среды окружения

Для настройки контейнера Certbot можно использовать следующие переменные среды:
| Переменная               | Описание                                                         | Значение по умолчанию |
|------------------------|---------------------------------------------------------------------|---------------|
| `CERTBOT_DOMAINS`      | Список доменов, разделенных запятыми, для которых необходимо получить сертификат  | - |
| `CERTBOT_EMAIL`        | Адрес электронной почты для уведомлений Let's Encrypt                       | - |
| `CERTBOT_KEY_TYPE`     | Тип закрытого ключа для генерации (ecdsa  или  rsa)                                     | `ecdsa` |
| `CERTBOT_SERVER`       | URL-адрес сервера ACME Staging: `https://acme-staging-v02.api.letsencrypt.org/directory` Production: `https://acme-v02.api.letsencrypt.org/directory`   | `https://acme-v02.api.letsencrypt.org/directory` |
| `CLOUDFLARE_API_TOKEN` | Токен API Cloudflare для аутентификации DNS (см. ниже, как его создать)                          | - |
| `CLOUDFLARE_CREDENTIALS_FILE` | Путь к файлу учетных данных Cloudflare. | `/cloudflare.ini` |
| `CLOUDFLARE_PROPAGATION_SECONDS` | Время ожидания (в секундах) после настройки TXT-записей DNS перед проверкой. Полезно, если обновление DNS происходит медленно.  | `10` |
| `DEBUG`                | Включить режим отладки (выводит дополнительную информацию на консоль)            | `false`                    |
| `PUID`                 | Идентификатор пользователя для запуска certbot                                        | `0`                    |
| `PGID`                 | Идентификатор группы для запуска certbot as                                        | `0`                    |
| `RENEWAL_INTERVAL`     | Интервал между проверками обновления сертификата в секундах. Установить в 0 чтобы отключить автоматическое продления и запустить только один раз.                          | 43200 (12 часов) |
| `REPLACE_SYMLINKS`     | Заменяет символические ссылки прямыми копиями файлов, на которые они ссылаются (требуется для Windows)  | `false`                    |
| `DEPLOY_HOOK` | скрипт, который запускается после успешного обновления SSL-сертификата | - |


### Создание API токена Cloudflare

> [!Warning]  
Относитесь к этому токену как к паролю. Он предоставит доступ к вашей учётной записи Cloudflare и может использоваться для изменения записей DNS. После создания токен показывается только однократно, сохраните его в надежном месте. 


1. Перейдите на страницу [Cloudflare API Tokens](https://dash.cloudflare.com/profile/api-tokens).
2. Нажмите "Create Token"
3. Нажмите "Use template" для шаблона "Edit Zone DNS".
4. Измените имя токена (необязательно)
5. Задайте конкретную зону в разделе "Zone Resources" (необязательно)
6. Нажмите "Continue to summary".
7. Нажмите "Create Token".

## Использование

1. Загрузите образ Docker:
   ```sh
   docker pull pdacity/acme-dns-certbot-with-hook:latest
   ```

2. Запустите контейнер с необходимыми переменными среды: 

> [!Caution]
> Обязательно замените `-v /path/to/your/certs:/etc/letsencrypt` на правильный путь на вашем хост-компьютере.

   ```sh
   docker run \
    -e CERTBOT_DOMAINS="domain.tld" \
    -e CERTBOT_EMAIL="your-email@domain.tld" \
    -e CLOUDFLARE_API_TOKEN="your-cloudflare-api-token" \
    -v /path/to/your/certs:/etc/letsencrypt \
   pdacity/acme-dns-certbot-with-hook:latest
   ```
> [!TIP]
> Для Wildcard-сертификатов используйте следующий порядок проверки healthcheck Docker: `domain.tld, *.domain.tld`

3. Контейнер автоматически сгенерирует и обновит сертификат. 

4. Если задан скрипт `DEPLOY_HOOK`, то после успешного обновления сертификата он будет выполнен.

## Ресурсы

- образ разработан на основе **[проекта](https://github.com/serversideup/docker-certbot-dns-cloudflare)** от  ServerSideUp.
