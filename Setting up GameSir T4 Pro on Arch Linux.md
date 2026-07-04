# Инструкция: Настройка GameSir T4 Pro (режим Xbox/XInput) на Arch Linux
![Arch Linux](https://shields.io/badge/Arch_Linux-Certified-1793D1?style=for-the-badge&logo=arch-linux&logoColor=white) ![Status](https://shields.io/badge/Status-Tested%20%26%20Working-success?style=for-the-badge&logo=github) ![License](https://shields.io/badge/License-MIT-yellow?style=for-the-badge)

Инструкция решает проблему, когда геймпад (через USB-донгл или кабель) ошибочно определяется как Nintendo Switch вместо Xbox 360 Controller.

## Шаг 1. Блокировка драйвера Nintendo на уровне ядра
Нужно запретить системе использовать встроенный модуль hid_nintendo, который принудительно включает режим Switch. 
> [!WARNING]
> Блокировка `hid_nintendo` отключит гироскоп и оригинальные геймпады Switch Pro.

В терминале создайте конфигурационный файл:
```
sudo nano /etc/modprobe.d/block-nintendo.conf
```
Вставьте в него следующие строки для жесткого запрета подгрузки драйвера:
```
blacklist hid_nintendo
install hid_nintendo /bin/true
```
Сохраните файл (`Ctrl+O`, `Enter`) и выйдите (`Ctrl+X`).


Обязательно обновите образ initramfs, чтобы изменения применились намертво при загрузке ПК:
```
sudo mkinitcpio -P
```
> [!IMPORTANT]
> Не забудьте обновить `initramfs`, иначе ядро продолжит загружать старый драйвер!


## Шаг 2. Создание агрессивного udev-правила.
Это правило автоматически перехватывает геймпад, даже если он аппаратно «забыл» настройки, и принудительно переводит его на стандартный Xbox-драйвер (xpad). 
Создайте файл правил автоматизации USB-портов:
```
sudo nano /etc/udev/rules.d/99-gamesir-xinput.rules
```
Вставьте в него эту строку:
```
SUBSYSTEM=="usb", ATTRS{idVendor}=="057e", ATTRS{idProduct}=="2009", ATTR{bInterfaceClass}=="03", RUN+="/sbin/modprobe xpad", RUN+="/bin/sh -c 'echo 057e 2009 > /sys/bus/usb/drivers/xpad/new_id'"
```
Сохраните файл (`Ctrl+O`, `Enter`) и выйдите (`Ctrl+X`).
Обновите правила udev в текущей сессии:
```
sudo udevadm control --reload-rules && sudo udevadm trigger
```

> [!CAUTION]
> Не редактируйте системные udev-правила без флага `sudo`, изменения не сохранятся.

## Шаг 3. Перезагрузка и проверка.
Перезагрузите компьютер. Вставьте USB-донгл и включите геймпад. Проверьте режим работы командой:
```
lsusb
```
Правильный результат: В списке устройств должно появиться Microsoft Xbox 360 Controller (или аналогичное упоминание Xbox/XInput).
> [!TIP]
> Команду `lsusb` можно запустить в режиме постоянного обновления: `watch -n1 lsusb`.
---

#### 💡 Шпаргалка на крайний случай
Если геймпад долго лежал без дела и полностью сбросил внутреннюю память, вставьте донгл, включите геймпад и зажмите кнопки `SELECT + START` на 5–7 секунд до вибрации. Это принудительно переключит режим внутри самого устройства.

---

> [!NOTE]
> Инструкция проверена на Arch Linux с ядром LTS.

