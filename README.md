# Развертывание TelegramBot в Docker и в Docker Compose

Данный репозиторий содержит пример запуска телеграм бота (https://github.com/DerekWestbrooke/TelegramBot.git) двумя способами:

- Отдельный контейнер Docker, в котором развертывается версия бота, использующая базу данных SQLite в качестве основного хранилища данных;
- Многоконтейнерное приложение при помощи Docker Compose, в котором развертывается версия бота, использующая базу данных PostgreSQL в качестве основного хранилища данных.

## Развертывание в Docker:
1) Для развертывания бота  в Docker необходимо в корне проекта создать Dockerfile, который представляет собой инструкцию по созданию Docker-image. В данном случае данный файл выглядит так:
```
# Official image python:3.12
FROM python:3.12

# Define a working directory
WORKDIR /TelegramBot

# Copy project files and folders to the working directory
COPY handlers/ handlers/
COPY parsers/ parsers/
COPY resources/ resources/
COPY main.py main.py
COPY requirements.txt requirements.txt

# Install all depencies from requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Launch a bot
CMD ["python", "main.py"]
```
В Dockerfile каждая строка имеет свое значение:
* ***FROM python:3.12*** - указывает базовый образ, т. е. в данном случае за основу взят образ ***python:3.12***;
* ***WORKDIR /TelegramBot*** - указывает на рабочую директорию, в которой и будет размещен проект в Docker;
* ***COPY handlers/ handlers/*** - копирует папку проекта ***handlers*** в рабочую директорию в Docker;
* ***COPY parsers/ parsers/*** - копирует папку проекта ***parsers*** в рабочую директорию в Docker;
* ***COPY resources/ resources/*** - копирует папку проекта ***resources*** в рабочую директорию в Docker;
* ***COPY main.py main.py*** - копирует главный файл проекта ***main.py*** в рабочую директорию в Docker;
* ***COPY requirements.txt requirements.txt*** - копирует файл необходимых зависимостей проекта ***requirements.txt*** в рабочую директорию в Docker;
* ***RUN pip install --no-cache-dir -r requirements.txt*** - запускает установку необходимых пакетов, перечисленных в файле ***requirements.txt***, при этом заранее отключив кеширование пакетов для уменьшения общего размера образа Docker;
* ***CMD ["python", "main.py"]*** - запуск команды при старте контейнера. В данном случае это запуск основного файла проекта ***main.py***.
2) Далее запускаем сборку образа на основании ранее созданного Dockerfile.
```
sudo docker build -t telegram_bot_image .
```
3) Проверяем создание docker image командой :
```
sudo docker images
```
4) Далее запускаем сборку docker container на основании docker image (в моем случае в целях повышения безопасности токен бота и API взаимодействий хранятся как переменные окружения, поэтому передача данных параметров необходимо осуществить с помощью параметра -e) командой:
```
sudo docker run -e TELEGRAM_BOT_TOKEN={your_bot_token} -e NEURAL_NETWORK_API_KEY={your_neural_network_api_key} -d --name telegram_bot telegram_bot_image
```
5) После выводим информацию о запущенных контейнерах (ID, название контейнера и изображения, команда запуска, время создания, статус, порт) и ищем свой контейнер:
```
sudo docker ps
```
6) Запускаем бота в телеграмме и видим, что бот успешно запущен.

## Развертывание в Docker Compose:  
1) Для развертывания бота  в Docker Compose необходимо в корне проекта создать docker-compose.yml, который содержит описание двух сервисов, необходимых для функционирования бота: сам бот и хранилище данных в виде базы данных PostgreSQL. В данном случае данный файл выглядит так:
```
version: '3.8'

services:
  db:
    image: postgres:14
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  bot:
    build: .
    restart: on-failure
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql+asyncpg://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      TELEGRAM_TOKEN: ${TELEGRAM_BOT_TOKEN}
    command: python main.py

volumes:
  pgdata:
```
В docker-compose.yml, как и в Dockerfile, каждая строка имеет свое значение:
* ***version: '3.8'*** - указывает версию синтаксиса для ***docker compose***;
* ***services:*** - указывает перечень сервисов (контейнеров);
* ***db:*** - конейнер, в котором будет развернута ***PostgreSQL***;
* ***image: postgres:14*** -  основа данного контейнера будет образ ***postgres:14*** с ***DockerHub***;
* ***restart: always*** - контейнер будет перезапускаться при сбоях в работе или при перезагрузке;
* ***environment: *** - далее будут перечислены переменные окржуения дял настройки базы данных;
* ***volumes:*** - монтрирование тома pgdata для хранения базы данных;
* ***ports:*** - проброс портов из контейнера на хост;
* ***bot:*** - контейнер для развертывания самого бота;
* ***environment:*** - далее будут перечислены переменные окржуения дял настройки базы данных;
* ***build: .*** - сборка образа бота по ***Dockerfile*** из пункта выше;
* ***restart: on-failure*** - перезапуск бота в случае ошибки;
* ***depends_on:*** - контейнер бота запускается только после контейнера базы данных;
* ***сommand: python main.py*** - команда, которая запускатся при старте контейнера;
* ***volumes:*** - объявление тома ***pgdata***.
2) Заносим значения переменных окружения в файл .env, который лежит в корне с проектом.
```
TELEGRAM_BOT_TOKEN=your_bot_api
POSTGRES_USER=your_postgres_user
POSTGRES_PASSWORD=your_postgres_password
POSTGRES_DB=your_postgres_db_name
```
3) Выполняем команду в папке с проектом, передав файл .env,в котором храняться все переменные окружения:
```
docker compose --env-file .env up -d
```
4) Проверяем логи:
```
docker compose logs
```
5) Если в них ошибок нет, запускааем бота и видим, что он успешно запущен.


  



