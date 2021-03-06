# FEECC Print Server

> Feecc Print Server — это микросервис, предназначенный для взаимодействия с принтером этикеток Brother серии QL. 
> Сервис предоставляет простой интерфейс REST API для печати изображений с опциональным текстовым комментарием. 
> Аутентификацию пользователей также поддерживается.
> 
> Микросервис является частью системы контроля качества Feecc — системы контроля качества с поддержкой Web3. 

Необходимое оборудование для рабочего места инженера (далее - РМИ) перечислено в [соответствующей статье](./workbench-and-components.md).
Исходный код модуля доступен по [сслыке](https://github.com/Multi-Agent-io/feecc-ipfs-gateway).

## Стек технологий

- Операционная система - [GNU/LINUX](https://www.gnu.org/), рекомендуется [Ubuntu](https://ubuntu.com/)
- Язык программирования - [Python](https://www.python.org/)
- ASGI-сервер - [Uvicorn](https://www.uvicorn.org/)
- REST-запросы - [FastAPI](https://fastapi.tiangolo.com/)
- Python3 HTTP-client - [httpx](https://www.python-httpx.org/)
- Сервис для публикации файлов на крупные ноды IPFS - [Pinata](https://www.pinata.cloud/)
- ПО для развертывания - [Docker](https://www.docker.com/)
- База данных - [MongoDB](https://www.mongodb.com/)

## Установка

Приложение должно запускаться в контейнере Docker и может быть настроено путем установки нескольких переменных окружения.

> Обратите внимание, что в этом руководстве предполагается хост Linux(Ubuntu), однако вы также можете запустить Print Server
> на любой другой ОС, но имейте в виду: часовой пояс определяется из файлов хоста `/etc/timezone` и `/etc/localtime` внутри 
> контейнера, которые не присутствуют на компьютерах с Windows, в таком случае внутри контейнера будет время UTC.

Установите [Docker](https://docs.docker.com/engine/install/) и [docker-compose](https://docs.docker.com/compose/install/).
Склонируйте репозиторий и измените переменные среды:
```
git clone https://github.com/Multi-Agent-io/feecc-print-server.git
cd feecc-print-server
vim docker-compose.yml
```

### Переменные среды

- `MONGODB_URI` — URI вашего подключения к MongoDB, заканчивающийся на `/db-name`.

- `PRODUCTION_ENVIRONMENT` - оставьте `null`, если вы хотите провести тестирование учетных данных для работы, иначе `True`.

- `PAPER_WIDTH`- Ширина ленты для печати в миллиметрах

- `PRINTER_MODEL`- Модель принтера этикеток.

- `RED`- *(true/null)* - Если используется черно-красная бумага.

```
sudo docker-compose up --build
```

Проверьте развертывание, перейдя по адресу http://127.0.0.1:8082/docs в браузере. Вы должны увидеть страницу спецификации SwaggerUI API.

## Описание модуля

### Эндпоинты

Работа модуля заключается в обрабортке запросов к эндпоинту:

###### `POST` /print_image  
> Распечатать файл на принтере, опционально можно аннотировать изображение текстом.

### Подмодули

Подмодуль `auth` отвечает за авторизацию пользователя, обращения к базе данных, а также за то, чтобы к камере был привязан
только один рабочий в каждый момент времени

Подмодуль `feecc_printer/Printer.py` отвечает непосредственно за взаимодействие с принтером:

###### `_address`
> Получение адреса принтера (адреса устройства в файловой системе).

###### `print_image`
> Печать изображения с опциональным аннотированием, вызывает `_get_image`, `_annotate_image`, `_print_image`.

###### `_get_image`
> Обработка и изменение размера изображения.

###### `_print_image`  
> Непосредственно отправка команды на печать принтеру.

###### `_annotate_image`  
> Добавить текстовую аннотацию снизу изображения.
