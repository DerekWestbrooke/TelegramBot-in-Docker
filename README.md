# Развертывание TelegramBot в Docker  
 
1) Для развертывания бота (https://github.com/DerekWestbrooke/TelegramBot.git) в Docker необходимо в корне проекта создать Dockerfile, который представляет собой инструкцию по созданию Docker-image. В данном случае данный файл выглядит так:
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



  



# TelegramBot-in-Docker
