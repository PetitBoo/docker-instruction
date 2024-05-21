# Создание Docker-образа и его развертывание на Ubuntu для начинающих
## Введение
Добро пожаловать в руководство по созданию и развертыванию своего первого образа на Docker для начинающих. Здесь мы поговорим о всех шагах, которые вас ожидают: от установки Docker до загрузки образа в сеть.
## Оглавление
1.	Установка Docker Engine
2.	Создание Docker-образа
3.	Запуск контейнера из образа
4.	Загрузка образа в сеть
## Установка Docker Engine
Первым делом мы должны удостовериться, что Docker работает на установленной версии Ubuntu. Запускаем команду:

` cat /etc/*rel*`

Находим строчку “VERSION” и сверяемся с [сайтом](https://docs.docker.com/engine/install/ubuntu/), что она работает:
 

Далее мы проверяем, что на машине нет установленных версий Docker, и удаляем их:

`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`

Создаем репозиторий для установки Docker:

```
# Добавляем официальный GPG-ключ Docker:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Добавляем репозиторий установки:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Далее нужно установить последнюю версию Docker при помощи команды:

` sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin `

Проверяем, что установка прошла успешно. Для этого мы запустим образ hello-world:

` sudo docker run hello-world `

Если установка завершена успешно, то вы увидите следующее:

##  Создание Docker-образа
Для начала мы создаем директорию, в которой будет храниться наш проект:

```
mkdir docker
mkdir docker/application
```

В папке docker/application создаем файл application.py:

```python
import http.server
import socketserver
import os
PORT = 8800
Handler = http.server.SimpleHTTPRequestHandler
# Create and listen at the HTTP socket, 
# dispatching the requests to the handler.
httpd = socketserver.TCPServer(("", PORT), Handler)
# Echo current status to the console and server logs. 
# It will be shown as the first line in the 'docker logs' command output.
os.system(f"echo 'serving at port {PORT}'")
# Handle requests until the server is terminated.
httpd.serve_forever()
```

В папке docker создаем Dockerfile:

```
# Используем образ Python с Docker Hub

FROM python:3.5 

# Добавь здесь комментарий по смыслу команды
WORKDIR ./quickstart_docker/docker/app

# И здесь комментарий
COPY ./application /app

# Комментарий
EXPOSE 80

# Комментарий
CMD ["python3", "/app/application.py"]
```
