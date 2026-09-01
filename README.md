# Домашка 15 - Elastic Stack (ELK)

Стек для сбора логов: Elasticsearch (ноды hot+warm), Logstash, Kibana и Filebeat в docker compose. Filebeat собирает логи всех docker-контейнеров и через Logstash складывает их в индексы `logstash-*`.

## Как запустить

```bash
cd elk
docker compose up -d
```

После старта:

- Kibana - http://localhost:5601
- Elasticsearch - http://localhost:9200

Контейнеров должно быть 5: `es-hot`, `es-warm`, `logstash`, `filebeat` и `kibana`.

### Тестовое приложение

Генерирует случайные логи (info/warning/error), чтобы было что смотреть в Discover:

```bash
docker run -d --name some_app --network elk_elastic \
  -v $(pwd)/pinger/run.py:/opt/run.py \
  library/python:3.9-alpine python3 /opt/run.py
```

Логи появятся в индексе `logstash-YYYY.MM.DD` - проверить можно так:

```bash
curl -s 'localhost:9200/_cat/indices/logstash-*'
```

## Что где лежит

- `docker-compose.yml` - стек
- `configs/` - конфиги logstash и filebeat
- `pinger/run.py` - dummy-приложение, которое шумит в stdout
- `screenshots/` - скриншоты docker ps, index patterns и Discover
- `solution.md` - текстовое решение

## Мелочи

- Нужен `vm.max_map_count` не меньше 262144 (`sysctl vm.max_map_count=262144`), иначе elasticsearch не стартует.
- Если logstash валится с `OutOfMemoryError` - подними `LS_JAVA_OPTS` в `docker-compose.yml`.
- В Kibana создай index-pattern по `logstash-*` (Data Views → Create) перед просмотром логов.