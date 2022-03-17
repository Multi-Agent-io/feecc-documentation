# HID Reader Daemon

> Легковесный python демон для отправки событий USB периферии на удалённые WEB сервера. Ивенты отправляются на 
> [feecc-workbench-daemon](./feecc-workbench-daemon.md)

Пример кода для отправки всех событий с **клавиатуры** на веб сервер [https://webhook.site](https://webhook.site/#!/595ddd9f-de34-4af8-845c-c52bb2614083):

```python
import asyncio
import requests
from EventToInternet.KeyboardListener import KeyboardListener


class BarcodeKeyboardListener(KeyboardListener):
    async def dict_handler(self, dict_event):
        print(dict_event)
        requests.post("https://webhook.site/595ddd9f-de34-4af8-845c-c52bb2614083", json=dict_event)


BarcodeKeyboardListener()

loop = asyncio.get_event_loop()
loop.run_forever()
```

## Стек технологий

- Операционная система - [Ubuntu Desktop](https://ubuntu.com/)
- Язык программирования - [Python](https://www.python.org/)
- Сервис - systemd/services

## Особенности

- Весь код асинхронный (даже в рамках 1 потока может работать параллельно).
- Каждое USB устройство распознаётся и отслеживается независимо, что позволяет подключать сколь угодно много устройств.
- Система поддерживает подключение и отключение устройств во время работы, настраиваясь на них автоматически. (В случае **raspberry pi zero w** это работает при подключении внешнего **USB-Hub**, так как она **Обязана** перезагрузиться, когда трогают её **micro-usb**, при этом продолжает работу если вставлять и вынимать устройства из этого **USB-Hub**)
- Используется python 3, что позволяет использовать данный код почти на каждом linux устройстве с минимумом действий для настройки проекта. 
- Система воспринимает любое устройство, которое можно читать как **USB** клавиатуру, включая **BarCode** сканеры.
- Программа способна отправлять числа, набранные с **верхнем и нижнем регистре** буквы и символы английского алфавита и символы **Numpad**.
- Смена раскладки отслеживает нажатие **LSHIFT** и **RSHIFT**, а также **CAPSLOCK** **(все клавиши настраиваются)** индивидуально для каждого устройства.
- Используется универсальная кодировка распознавания  **ASCII** **(кодировка настраивается)**.
- Сообщение отправляется по нажатию клавиши **ENTER** или её же на **Numpad** **(все клавиши настраиваются)**.
- Индивидуальный буфер сообщения ограничен **128** символами **(длина настраиваются)**, и после его переполнения, в памяти для этого устройства ввода будут храниться **128** последних набранных символов.
- Сервис собирает и выводит дополнительную информацию об устройстве для его идентификации и анализа трафика.

## Установка

```bash
sudo apt update
sudo apt upgrade
sudo apt install git
sudo apt install htop python3 python3-dev python3-pip gcc
cd ~
git clone https://github.com/Multi-Agent-io/feecc-hid-reader-daemon
sudo mv feecc-hid-reader-daemon /etc/systemd/system/
sudo chown -R root:root /etc/systemd/system/feecc-hid-reader-daemon
cd /etc/systemd/system/feecc-hid-reader-daemon
sudo python3 -m pip install -r requirements.txt
sudo bash install.sh
```

## Удаление

```bash
sudo bash /etc/systemd/system/feecc-hid-reader-daemon/uninstall.sh

sudo rm -rf /etc/systemd/system/feecc-hid-reader-daemon*
```

## Персонализация

Файл **EventToInternet/\_\_init\_\_.py** - Содержит константы для работы с USB периферией.

```python
"""
    Максимальная длина строки в символах.
    Если вводится строка длиннее, чем из {KEYBOARD_MAX_STRING_LENGTH} символов, то старые символы строки сообщения сотрутся,
    и система будет помнить только последние {KEYBOARD_MAX_STRING_LENGTH} символов.
"""
KEYBOARD_MAX_STRING_LENGTH = 128  # 128 - Максимальная длина штрих кода согласно GS1-128

"""
    Период автоматического обновления списка всех подключенных usb устройств в секундах.
    Каждые {KEYBOARD_UPDATE_DEVICES_TIMEOUT} система будет смотреть, подключили ли новую клавиатуру или сканер через USB порт.
"""
KEYBOARD_UPDATE_DEVICES_TIMEOUT = 1.0  # Время запуска сканера 3 секунды, поэтому ждать ещё 1 секунду сверх этого приемлемо
```

Файл **EventToInternet/\_config\_event\_to\_string.py** - Содержит выбранную кодировку для интерпретации **Событий Клавиатуры** как **Символы**, а также список клавиш смены **Капитализации** и **Триггеры Отправки**.

```python
# Кодировка нижнего регистра букв
regular_letters_codes = {
    16: u'q', 17: u'w', 18: u'e', 19: u'r', 20: u't', 21: u'y', 22: u'u', 23: u'i', 24: u'o', 25: u'p', 30: u'a',
    31: u's', 32: u'd', 33: u'f', 34: u'g', 35: u'h', 36: u'j', 37: u'k', 38: u'l', 44: u'z', 45: u'x', 46: u'c',
    47: u'v', 48: u'b', 49: u'n', 50: u'm',
}

# Кодировка нижнего регистра символов
regular_symbols_codes = {
    2: u'1', 3: u'2', 4: u'3', 5: u'4', 6: u'5', 7: u'6', 8: u'7', 9: u'8', 10: u'9', 11: u'0', 12: u'-', 13: u'=',
    26: u'[', 27: u']', 39: u';', 40: u'\'', 41: u'`', 43: u'\\', 51: u',', 52: u'.', 53: u'/', 57: u' '
}

# Кодировка верхнего регистра букв
capital_letters_codes = {
    16: u'Q', 17: u'W', 18: u'E', 19: u'R', 20: u'T', 21: u'Y', 22: u'U', 23: u'I', 24: u'O', 25: u'P', 30: u'A',
    31: u'S', 32: u'D', 33: u'F', 34: u'G', 35: u'H', 36: u'J', 37: u'K', 38: u'L', 44: u'Z', 45: u'X', 46: u'C',
    47: u'V', 48: u'B', 49: u'N', 50: u'M',
}

# Кодировка верхнего регистра символов
capital_symbols_codes = {
    2: u'!', 3: u'@', 4: u'#', 5: u'$', 6: u'%', 7: u'^', 8: u'&', 9: u'*', 10: u'(', 11: u')', 12: u'_', 13: u'+',
    26: u'{', 27: u'}', 39: u':', 40: u'\"', 41: u'~', 43: u'|', 51: u'<', 52: u'>', 53: u'?', 57: u' ',
}

# Кодировка Numpad
numpad_symbols_codes = {
    79: u'1', 80: u'2', 81: u'3', 75: u'4', 76: u'5', 77: u'6', 71: u'7', 72: u'8', 73: u'9', 82: u'0', 98: u'/',
    55: u'*', 74: u'-', 78: u'+', 83: u'.'
}

# Клавиши - Триггеры отправки сообщения. При нажатии на эту клавишу отправляется текущая версия сообщения,
# после чего строка обнуляется, и устройство снова ожидает ввод данных.
send_trigger_keys = {"KEY_ENTER", "KEY_KPENTER"}

# Клавиши для временной смены (при нажатии изменить, при отпускании
# вернуть) капитализации символов клавиатуры. Обычно Shift.
capitalize_all_keys = {"KEY_LEFTSHIFT", "KEY_RIGHTSHIFT"}

# Клавиши для фиксированной смены капитализации символов клавиатуры. Обычно CapsLock.
capitalize_symbols_turn_keys = {"KEY_CAPSLOCK", }
```
