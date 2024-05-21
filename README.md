# Основы Docker. Создание образа и запуск контейнера
## Введение
Добро пожаловать в руководство по созданию образа и запуска контейнера на Docker для начинающих. Здесь мы поговорим о всех шагах, которые вас ожидают: от установки Docker до загрузки образа в сеть. Руководство написано для Ubuntu.
## Оглавление
1.	Установка Docker Engine
2.	Создание Docker-образа
3.	Запуск контейнера и основные команды Docker
4.	Загрузка образа в сеть
## Установка Docker Engine
Первым делом мы должны удостовериться, что Docker работает на установленной версии Ubuntu. Запускаем команду:

```
cat /etc/*rel*
```

Находим строчку `VERSION` и сверяемся с [сайтом](https://docs.docker.com/engine/install/ubuntu/), что она работает:
![Пример](./images/version.jpg)

 

Далее мы проверяем, что на машине нет установленных версий Docker, и удаляем их:

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Создаем репозиторий для установки Docker:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Далее нужно установить последнюю версию Docker при помощи команды:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Проверяем, что установка прошла успешно. Для этого мы запустим образ `hello-world`:

```
sudo docker run hello-world
```

Если установка завершена успешно, то вы увидите следующее:

##  Создание Docker-образа
Для начала мы создаем директории, в которой будет храниться наш проект:

```
mkdir docker
mkdir docker/application
```

В **docker/application/** мы будем хранить файлы для нашего образа. Создаем в ней файл `application.py`:

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

В директории **docker/** создаем `Dockerfile` и прописываем в нем следующие зависимости:

```Docker
# Используем образ Python с Docker Hub

FROM python 

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем все нужные файлы в директорию, где будет собран образ
COPY ./application /app

# Открываем порт
EXPOSE 8800

# Запускаем приложение
CMD ["python", "/app/application.py"]
```

Создаем образ при помощи команды:

```
docker build -t exampleapp -f Dockerfile .
```

`-t` используется для явного указания названия образа.
`exampleapp` – название образа, которое мы использовали.
`-f` используется для явного указания Dockerfile.
`.` – папка, в которой нужно искать Dockerfile.

Чтобы проверить, что появилась новая папка, в которой хранится наш образ, можно запустить команду:

```
ls -l
```

## Запуск контейнера и основные команды Docker
Запускаем образ при помощи команды:

```
docker run -p 8800:8800 -d exampleapp
```

Чтобы посмотреть список контейнеров, пишем:

```
docker ps -a
```

`-a` нужен, чтобы вывести все контейнеры, в том числе остановленные.

Чтобы остановить контейнер, пишем:

```
docker stop < название или ID контейнера>
```

Чтобы удалить контейнер, используйте команду:

```
docker rm -f <название или ID контейнера>
```

Чтобы посмотреть все доступные на хосте образы, используем:

```
docker images
```

При помощи этой команды удаляем образы:

```
docker rmi <название образа>
```

Важно! Перед удалением образа, нужно остановить все контейнеры, которые его используют.

## Загрузка образа в сеть

Сначала нужно зарегистрироваться на [Docker Hub](https://hub.docker.com/).
Далее создаем образ, в котором в качестве названия будет использован наш логин:

```
docker build -t <логин>/<название образа>
```

Прописываем команду:

```
docker login
```

Затем вводим свой логин и пароль.
Чтобы добавить свой образ на Docker Hub, вводим:

```
docker push <логин>/<название образа>
```
