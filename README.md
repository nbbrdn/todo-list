# ToDo List

Run the interactive bash-session inside the container

```bash
docker-compose run --rm --service-ports api /bin/bash
```

## From the Telegram group discussion

ModuleNotFoundError: No module named 'psycopg2' - очевидно требуется установить модуль psycopg2, (мы в практике это пропустили, считаем что решение можно найти самостоятельно)
Навскидку -
Зайти в баш сессию в контейнере `docker-compose run —rm api bash`
в это bash сессии проверяем `python -c "import psycopg2"` - если не работает, значит действительно не установлен
устанавливаем `poetry add psycopg2-binary` - если ошибки - разбираемся. По результатам в pyproject.toml должна появиться запись о том что нашему проекту необходим psycopg2-binary и еще поставится сам модуль куда-то в недра системы в site-packages питона.
После этого проверяем снова `python -c "import psycopg2"` - работает - все хорошо.
Дальше есть такая тонкость - когда вы что-то запускали будучи в контейнере изменения сохраняются в файлах /app благодаря нашим вольюмам (конкретно в случае установки пакета в pyproject.toml), но изменения в других частях файловой системы (всякие site-packages питона) пропадут.
Поэтому перед следующим запуском надо ребилднуть контейнер `docker-compose build api` , заново исполнится/применится Dockerfile и на этапе RUN poetry install поэтри поставит все что положено по инструкциям из pyproject.toml (если точнее из poetry.lock), но на это раз уже перманентно в соответствующий слой образа (мы ведь помним что каждая команда RUN создает слой в докер-образе?)
Скорее всего проблема в этом.
