# nginx_proxy

Минимальный `Nginx` reverse proxy для Docker-сети `proxy_net`.

## Что делает

- принимает входящий HTTP-трафик на `80`
- маршрутизирует домены на внутренние Docker-сервисы
- не публикует backend-контейнеры наружу
- использует Docker DNS resolver, чтобы не падать при смене IP контейнеров

## Маршрутизация

- `antifraud.pp.ua` -> `antifraud_app:8000`
- `fsprojects.pp.ua` -> `fsprojects_app:8000`
- `assistant.fsprojects.pp.ua` -> `assistant_app:8000`

## Архитектура

```text
Internet / Cloudflare
        |
        v
   nginx_proxy :80
        |
        v
   Docker network: proxy_net
      |- antifraud_app:8000
      |- fsprojects_app:8000
      `- assistant_app:8000
```

`nginx.conf` использует:

- `resolver 127.0.0.11 valid=10s ipv6=off;`
- `proxy_pass http://$upstream;`

Это позволяет:

- не привязываться к старому IP контейнера после recreate
- не ронять весь proxy из-за статического upstream resolution на старте

## Состав

- [`docker-compose.yml`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/docker-compose.yml) - контейнер `nginx`
- [`nginx.conf`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/nginx.conf) - домены и proxy rules
- [`instruction.md`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/instruction.md) - быстрые команды для запуска и дебага

## Запуск

```bash
docker network create proxy_net
cd /home/drhtka/projects/nginx_proxy
docker compose up -d
```

## Проверка

```bash
docker compose ps
docker compose logs --tail=50 nginx
curl -I -H 'Host: assistant.fsprojects.pp.ua' http://127.0.0.1/health
curl -I -H 'Host: fsprojects.pp.ua' http://127.0.0.1/
```
