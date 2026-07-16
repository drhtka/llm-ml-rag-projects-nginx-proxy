## Создать сеть

```bash
docker network create proxy_net
```

## Запустить проект

ssh -i /Users/drhtka/Downloads/Projects/Llm_ml_RAG/fsprojects_ssh/id_ed25519 drhtka@91.238.104.198

```bash
cd /home/drhtka/projects/anti_fraud_analytics_platform && docker compose up -d
cd /home/drhtka/projects/nginx_proxy && docker compose up -d
```

## Дебаг

```bash
cd /home/drhtka/projects/nginx_proxy && docker compose logs -f nginx
cd /home/drhtka/projects/anti_fraud_analytics_platform && docker compose logs -f app

cd /home/drhtka/projects/nginx_proxy && docker compose logs --tail=20 -f nginx
cd /home/drhtka/projects/anti_fraud_analytics_platform && docker compose logs --tail=20 -f app


cd /home/drhtka/projects/anti_fraud_analytics_platform && docker compose ps
cd /home/drhtka/projects/nginx_proxy && docker compose ps
```

## Быстрая проверка

```bash
curl -I http://antifraud.pp.ua
curl http://antifraud.pp.ua/health
```

```bash
cd /home/drhtka/projects/anti_fraud_analytics_platform && docker compose up -d
cd /home/drhtka/projects/nginx_proxy && docker compose up -d
```
