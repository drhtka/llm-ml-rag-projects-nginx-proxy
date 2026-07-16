# nginx_proxy

Минимальный `Nginx` reverse proxy для Docker-сети `proxy_net`, который маршрутизирует внешние домены на внутренние Docker-сервисы проектов.

## Что делает

- принимает входящий HTTP-трафик на `80`
- маршрутизирует домены на внутренние Docker-сервисы
- не публикует backend-контейнеры наружу
- использует Docker DNS resolver, чтобы не падать при смене IP контейнеров

## Маршрутизация

- `antifraud.pp.ua` -> `antifraud_app:8000`
- `fsprojects.pp.ua` -> `fsprojects_app:8000`
- `assistant.fsprojects.pp.ua` -> `assistant_app:8000`

## Боевые URL

- `https://antifraud.pp.ua/`
- `https://fsprojects.pp.ua/`
- `https://assistant.fsprojects.pp.ua/`

## Архитектура

```text
Internet / Cloudflare
        |
        v
   nginx_proxy :80
        |
        v
   Docker network: proxy_net
      |- anti_fraud_analytics_platform
      |    `- antifraud_app:8000
      |
      |- fsprojects
      |    `- fsprojects_app:8000
      |
      `- ai_knowledge_assistant
           `- assistant_app:8000
```

## Архитектура Nginx

`nginx_proxy` сам не хранит бизнес-логику и не обслуживает приложение напрямую. Он делает только edge-routing:

1. принимает запрос от Cloudflare или напрямую снаружи;
2. смотрит на `Host`;
3. выбирает нужный upstream внутри `proxy_net`;
4. пробрасывает заголовки `Host`, `X-Forwarded-*`, `Upgrade`, `Connection`;
5. отдаёт ответ клиенту от соответствующего backend-сервиса.

По сути схема такая:

```text
Client
  -> Cloudflare
    -> nginx_proxy
      -> antifraud_app
      -> fsprojects_app
      -> assistant_app
```

`nginx.conf` использует:

- `resolver 127.0.0.11 valid=10s ipv6=off;`
- `proxy_pass http://$upstream;`

Это позволяет:

- не привязываться к старому IP контейнера после recreate
- не ронять весь proxy из-за статического upstream resolution на старте
- держать `ai_knowledge_assistant`, `fsprojects` и `antifraud` развязанными друг от друга на уровне запуска proxy

## Состав

- [`docker-compose.yml`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/docker-compose.yml) - контейнер `nginx`
- [`nginx.conf`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/nginx.conf) - домены и proxy rules
- [`instruction.md`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/nginx_proxy/instruction.md) - быстрые команды для запуска и дебага

Связанный backend-проект:

- [`ai_knowledge_assistant`](file:///Users/drhtka/Downloads/Projects/Llm_ml_RAG/ai_knowledge_assistant)

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
