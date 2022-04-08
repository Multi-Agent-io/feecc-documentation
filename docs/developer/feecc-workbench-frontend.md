# Workbench Frontend

> Веб-приложение для системы мниторинга производства на базе платформы Feecc. Является клиентской стороной бэкенда
> [`feecc_workbench_daemon`](./feecc-workbench-daemon.md).


## Стек технологий

- Операционная система неважна.
- Язык программирования - [JavaScript](https://www.javascript.com/)
- Библиотека для неизменных переменных - [Immutable](https://immutable-js.com/)
- Библиотека для разработки пользовательских интерфейсов - [React](https://reactjs.org/)
- Библиотека для управления состянием приложения - [Redux](https://redux.js.org/)
- Сборщик модулей JavaScript - [Webpack](https://webpack.js.org/)
- ПО для развертывания - [Docker](https://www.docker.com/)

## Установка

Ниже - два способа установки приложения

### Установка с помощью `npm`

- Установите Node и `npm` с [официального сайта](https://nodejs.org/en/) или с помощью [`nvm`](https://github.com/nvm-sh/nvm).
- Склонируйте репозиторий и установите зависимости (код ниже для GNU/Linux):
```bash
git clone https://github.com/Multi-Agent-io/feecc-workbench-frontend
cd feecc-workbench-frontend
sudo npm install
```

Для запуска в режиме разработчика необходимо выполнить команду `npm run dev-server`.

Для подготовки приложения к разворачиванию необходимо выполнить команду `npm run build`. После этого в корневой папке 
проекта появится папка `target`. Там будут все необходимые для рабты файлы. Корневым файлом `html` является `index.html`.

### Установка с помощью Docker

Установите [Docker](https://docs.docker.com/engine/install/) и [docker-compose](https://docs.docker.com/compose/install/).
Установите и запустите приложение:
```bash
git clone https://github.com/Multi-Agent-io/feecc-workbench-frontend
cd feecc-workbench-frontend
docker-compose build
docker-compose up
```

## Описание приложения

### Конфигурация

Переменные конфигурации приложения расположены в файле `configs/config.js`. В той же папке `configs` расположен 
файл `feecc_frontend.service`, в котором описан `systemd` сервис для автоматического начала работы приложения при старте 
хоста.

### Перевод

Для перевода используется модуль `i18next` и загрузчик для переводов из файлов `.csv`. Переводы хранятся в 
`public/translation.csv`. Первую строку этого файла, нельзя изменять без изменений в `i18next.js`!

### Базовые стили

Цветовая палитра лежит в файле `src/index.scss`.

### Redux

Все данные Redux store хранятся в `src/reducers`. Основной файл компоновки и импорта store - `src/reducers/main.js`. 
В файле `src/reducers/common.js` хранятся типы запросов к store, `fetchWrapper` и `axiosWrapper`. Эти две функции 
выполняют все запросы к бэкенду. Функция `reportError` существует для удобства разработки и отлова ошибок с Redux store.

Все записи в store производятся в `src/reducers/stagesReducer.js` касательно стадий производства и в 
`src/reducers/RevisionsReducer.js` касательно ревизии.

Все запросы к бэкенду и store обрабатываются в `src/reducers/stagesActions.js`касательно стадий производства и в 
`src/reducers/RevisionsActions.js` касательно ревизии.

### Компоненты

Всё заполнение подгружается в `src/components/App.js` в зависимости от состояния `store` и `pathname`. `pathname` будет динамически 
меняться в процессе работы, но это не даёт возможности свободно перемещаться по этапам, так как восстановление сессии, 
описанное в `/src/components/Composition/Composition.js::fetchComposition`, вернёт пользователя на текущий этап 
производства, если оно идёт.

#### Composition.js

Большая часть логики лежит в `/src/components/Composition/Composition.js`. Вся логика согласована с [`feecc_workbench_daemon`](./feecc-workbench-daemon.md).

#### GatherComponents.js

Компонента для отображения списка просканированных/не просканированных элементов для финальной сборки (у каждой сборки они свои и отображаются именно тут).

#### Header.js

Компонента для хранения части логики работы перемещения по страницам и шапки страницы.

#### Login.js

Компонента с заглушкой и опросом сервера на наличие авторизации пользователя.

#### Menu.js

Компонента с минимумом логики и двумя кнопками для начала сборки и завершения сессии.

#### Notifications.js

Компонента для отображения всплывающих уведомлений.

#### RevisionsTracker.js

Компонента для выдвигающегося окошка со списком изделий, которые отправили на доработку.

#### Stopwatch.js

Компонента для отображения секундомеров для каждого этапа производства.

