## Вводные

**30.05.2024 DockerHub перестал работать в РФ. При попытке обратить к нему начала возвращаться ошибка 403**
```
Since Docker is a US company, we must comply with US export control regulations. In an effort to comply with these, we now block all IP addresses that are located in Cuba, Iran, North Korea, Republic of Crimea, Sudan, and Syria. If you are not in one of these cities, countries, or regions and are blocked, please reach out to https://hub.docker.com/support/contact/
```

На данный момент работает механизм зеркалирования репозитория (registry:2). 

Вы можете развернуть зеркало на любом зарубежном VPS и использовать его в обход блокировки.

Я использую хостинг [Server space](https://serverspace.ru/ref/565356), ДЦ Амстердам. Единственный из более-менее нормальный хостеров с ДЦ за рубежом и поддержкой Российских карт.

## Сервисы

В docker-compose файле описана следующая конфигурация:
- [Traefik](https://doc.traefik.io/traefik/) с настроенным HTTPS (ACME, Let's Encrypt) и [фильтром IP адресов](https://doc.traefik.io/traefik/middlewares/http/ipallowlist/)
- [Docker registry](https://hub.docker.com/_/registry)

Traefik выступает в качестве фронтенда перед докер реестром.

## Настройка
### Сервис traefik
#### Базовая настройка
```text
- "--certificatesresolvers.myresolver.acme.email={your@email}"
```
Замените  {your@email} на свой адрес электронной почты

#### Открытие доступа в веб интерфейс traefik
Приведите аргументы команды к таким значениям:
```text
- "--log.level=DEBUG"
- "--accesslog=true"
- "--api.insecure=true"
- "--api.dashboard=true"
- "--api.debug=true"
```
Раскоментируйте строку с портом
```text
- "8080:8080"
```
Перезапустите traefik. Теперь вы можете зайти в веб интерфейс по адресу http://{IP}:8080

### Сервис dockerproxy
```text
- "traefik.http.routers.dockerproxy.rule=Host(`{domain}`)"
```
Замените {domain} на публичный адрес сервиса (на этот адрес будет выпущен SSL сертификат)

```text
- "traefik.http.middlewares.ipallowlist.ipallowlist.sourcerange=127.0.0.1/32"
```
Пропишите в этом блоке белый список IP адресов, или удалите следующие строчки для отключения фильтра
```text
- "traefik.http.middlewares.ipallowlist.ipallowlist.sourcerange=127.0.0.1/32"
- "traefik.http.routers.dockerproxy.middlewares=ipallowlist"
```

## Запуск 
```text
docker compose up -d
```

### Настройка Docker Desktop
Заходим Settings -> Docker Engine

Добавляем в JSON такую запись (не забудте заменить {domain} на публичный адрес своего инстанса docker registry)
```text
"registry-mirrors": [
    "https://{domain}"
]
```

В результат должно получится примерно так:
```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://{domain}"
  ]
}
```
