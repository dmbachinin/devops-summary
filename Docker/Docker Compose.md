# Docker Compose

```plantext
    В данном файле собрана информация по работе с Docker Compose
```

- [Docker Compose](#docker-compose)
  - [Docker Compose базовая инофрмация](#docker-compose-базовая-инофрмация)
  - [Разница между Dockerfile и Docker Compose](#разница-между-dockerfile-и-docker-compose)
  - [Структура файла docker-compose.yml](#структура-файла-docker-composeyml)
    - [volumes](#volumes)
      - [Как работает связывание директорий](#как-работает-связывание-директорий)
    - [Дополнительные опции](#дополнительные-опции)

## Docker Compose базовая инофрмация

`Docker Compose` — это инструментальное средство, входящее в состав `Docker`. Оно предназначено для решения задач, связанных с развёртыванием проектов

Как узнать, нужно ли вам, при развёртывании некоего проекта, воспользоваться `Docker Compose`? На самом деле — очень просто. Если для обеспечения функционирования этого проекта используется несколько сервисов, то `Docker Compose` может вам пригодиться. Например, в ситуации, когда создают веб-сайт, которому, для выполнения аутентификации пользователей, нужно подключиться к базе данных. Подобный проект может состоять из двух сервисов — того, что обеспечивает работу сайта, и того, который отвечает за поддержку базы данных

## Разница между Dockerfile и Docker Compose

`Dockerfile и Docker Compose` — это два разных инструмента, используемых в Docker для управления контейнерами, и каждый из них решает свои задачи. Вот их основные отличия:

`Dockerfile` нужен для создания образа приложения, тогда как `Docker Compose` — для одновременного управления и запуска нескольких контейнеров с настройками их взаимодействий. Эти два инструмента часто используются вместе: Dockerfile создаёт образ, который затем можно использовать в `docker-compose.yml` для запуска приложений.

**Основные различия:**

| **Dockerfile**                              | **Docker Compose**                         |
|---------------------------------------------|--------------------------------------------|
| Используется для сборки одного образа Docker | Используется для управления несколькими контейнерами |
| Описывает, как собрать образ приложения     | Описывает, как запустить несколько контейнеров и как они взаимодействуют |
| Файл инструкций по сборке (обычно один)     | Файл конфигурации для множества сервисов   |
| Команда: `docker build` для сборки образа   | Команда: `docker-compose up` для запуска всех сервисов |

## Структура файла docker-compose.yml

```yaml

# Файл docker-compose должен начинаться с тега версии.
version: '3.8' # Указывает версию формата Docker Compose. Разные версии могут поддерживать различные функции.

# Раздел, в котором будут описаны сервисы, начинается с 'services'.
# 1 сервис = 1 контейнер.
# Сервисом может быть клиент, сервер, сервер баз данных...
services:

  web: # Название сервиса (Контейнера)
    build:  # Определяет, как собрать образ для сервиса
      context: ./web # Директория, содержащая Dockerfile и все необходимые файлы для сборки
      args: # Позволяет передавать аргументы на этапе сборки образа. Эти аргументы можно использовать в Dockerfile
        NODE_VERSION: 14
      dockerfile: Dockerfile #  Опционально указывает имя Dockerfile (по умолчанию Dockerfile)

    ports: # Открывает порты для взаимодействия с контейнером
      - "8080:80" # Формат: "host_port:container_port". Например, "8080:80" направляет трафик с порта 8080 на хосте к порту 80 в контейнере

    environment: # Задает переменные окружения для контейнера. Они могут использоваться приложением внутри контейнера
      NODE_ENV: production # Можно задавать как в виде списка, так и в виде ключ-значение

    volumes: # Позволяет монтировать директории или файлы из хостовой системы в контейнер, что позволяет сохранять данные между перезапусками контейнера или делать разработку более удобной
      - ./web:/usr/src/app # Формат: "host_path:container_path". Например, - ./web:/usr/src/app монтирует директорию ./web с хоста в /usr/src/app внутри контейнера

    networks: # Позволяет определить пользовательские сети, которые используются для связи между контейнерами. По умолчанию все контейнеры подключаются к одной сети, но можно создать и использовать собственные
      - frontend
    depends_on: # Определяет порядок запуска контейнеров. Например, если сервис app зависит от сервиса db
      - db # Однако это не гарантирует, что db будет готов к работе, когда app запустится

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:

volumes:
  db_data:

```

### volumes

Монтирование в Docker с помощью `volumes` позволяет связывать директории или файлы на вашем хосте с директориями внутри контейнера. Это обеспечивает возможность совместного использования данных между контейнерами и хостовой системой, а также позволяет сохранять данные между перезапусками контейнера

Связывание директорий при монтировании в Docker происходит таким образом, что содержимое директории на хосте и содержимое директории в контейнере пересекаются, и это может приводить к разным результатам в зависимости от того, как вы настроите монтирование.

#### Как работает связывание директорий

1. **Основные правила**:
   - Когда вы монтируете директорию из хоста в контейнер, содержимое директории на хосте "перекрывает" содержимое директории в контейнере.
   - То есть, если в директории на хосте есть файлы, они будут видны внутри контейнера, а те файлы, которые уже есть в контейнере в монтируемой директории, будут недоступны, пока монтирование активно.

2. **Поведение при монтировании**
   - Если директория на хосте уже содержит файлы, они становятся доступными внутри контейнера, и файлы из контейнера, которые находятся в той же директории, не будут видны.
   - Если директория на хосте пустая, при монтировании она будет отображать содержимое контейнера.

**Что происходит с данными?**

- **Добавление данных**: Если вы добавите файлы в монтируемую директорию внутри контейнера, они будут добавлены в директорию на хосте. Например, если в контейнере вы создадите файл в `/container/data`, он появится в `/host/data`.

- **Существующие данные**: Если вы хотите сохранить данные в контейнере и при этом иметь доступ к данным на хосте, вам нужно быть осторожным с тем, какие данные вы храните в монтируемой директории.

Монтирование директорий обеспечивает **двустороннюю синхронизацию** данных между контейнером и хостом: изменения в одной стороне будут отражены и на другой. Это делает работу с данными более гибкой и удобной

**Пример 1: Монтирование с данными на хосте:**

- **Содержимое директории на хосте (`/host/data`)**:
  
  ```plantext
  file1.txt
  file2.txt
  ```

- **Содержимое директории в контейнере (`/container/data`)**:
  
  ```plantext
  file3.txt
  ```

- **Результат**:
  - Внутри контейнера вы увидите:
  
    ```plantext
    file1.txt
    file2.txt
    ```

  - Файл `file3.txt` будет недоступен в контейнере, поскольку директория монтируется с хоста.

**Пример 2: Монтирование с пустой директорией на хосте:**

- **Содержимое директории на хосте (`/host/data`)**:

  ```plantext
  (пусто)
  ```

- **Содержимое директории в контейнере (`/container/data`)**:
  
  ```plantext
  file3.txt
  ```

- **Результат**:
  - Внутри контейнера вы увидите:

    ```plantext
    file3.txt
    
    ```

  - Поскольку директория на хосте была пустой, содержимое контейнера остается доступным.

### Дополнительные опции

```yaml

healthcheck: # Позволяет задавать проверки состояния контейнера. Это может быть полезно для определения, работает ли сервис корректно
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 30s
  timeout: 10s
  retries: 3

labels: # Позволяет добавлять метаданные к контейнерам в виде пар ключ-значение. Это может быть полезно для организации и управления контейнерами
  com.example.description: "My app"
```
