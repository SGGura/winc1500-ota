# WINC1500 OTA firmware (3A0)

Репозиторий хранит OTA-образ для обновления модуля **WINC1500** на приборе **iEMAT1** (InspeCore) по Wi‑Fi.

| Файл | Описание |
|------|----------|
| `m2m_ota_3a0.bin` | OTA image, ~236 KB, прошивка **19.7.10** |
| `version.txt` | Версия образа и дата публикации |

**Прямая ссылка для WINC (в прошивке iEMAT1, &lt; 100 символов):**

```
https://raw.githubusercontent.com/SGGura/winc1500-ota/main/m2m_ota_3a0.bin
```

Источник бинарника: [Microchip-MPLAB-Harmony/wireless_wifi](https://github.com/Microchip-MPLAB-Harmony/wireless_wifi).

Скрипты прошивки и публикации лежат в проекте iEMAT1:

`D:\MCS_PIC\InspeCore\iEMAT1\Scripts\`

---

## OTA по Wi‑Fi (на приборе)

После публикации образа на GitHub на UART iEMAT1:

```
wifi ota <ssid> <password>
```

Пример:

```
wifi ota GURAWORK GURA03071963
```

URL прошивки зашит в `My_WiFi.h` (`WIFI_OTA_URL`). Имя файла в команде не указывается.

---

## publish_ota_github.bat — обновить этот репозиторий

Скачивает последнюю прошивку Microchip, собирает OTA-образ и пушит в [SGGura/winc1500-ota](https://github.com/SGGura/winc1500-ota).

```bat
cd /d D:\MCS_PIC\InspeCore\iEMAT1\Scripts
publish_ota_github.bat
```

**Что делает:**

1. Загрузка с GitHub [wireless_wifi](https://github.com/Microchip-MPLAB-Harmony/wireless_wifi) (ветка `master`).
2. Сборка `ota_firmware\m2m_ota_3A0.bin` (`prepare_image.cmd`).
3. Копия в `SGGura-winc1500-ota` и `git push` на GitHub.

**Опции:**

| Команда | Назначение |
|---------|------------|
| `publish_ota_github.bat` | Полный цикл: Microchip → сборка → GitHub |
| `publish_ota_github.bat --no-fetch` | Только push (образ уже собран) |
| `publish_ota_github.bat --no-push` | Только локальные файлы, без Git |
| `publish_ota_github.bat --force` | Принудительно перекачать Microchip |
| `publish_ota_github.bat -v` | Подробный лог |

**Требования:** Windows, Python 3, Git ([git-scm.com](https://git-scm.com/download/win)), доступ к GitHub. В `Scripts\config.bat` при необходимости:

```bat
set "GIT_EXE=C:\Program Files\Git\cmd\git.exe"
```

После смены версии прошивки на Microchip проверьте совпадение драйвера в iEMAT1 (`m2m_types.h`, `M2M_RELEASE_VERSION_*`), иначе возможна ошибка **-13**.

---

## flash_winc.bat — прошивка WINC по UART

Прямая прошивка модуля с ПК (без Wi‑Fi OTA). Тот же пакет Microchip в папке `Scripts`.

```bat
cd /d D:\MCS_PIC\InspeCore\iEMAT1\Scripts
flash_winc.bat 9
```

`9` — номер COM-порта USB-UART к WINC.

**Порядок:**

1. (если без `--no-fetch`) скачивание свежей прошивки с GitHub Microchip;
2. `download_all.bat UART 1500 0 9` — прошивка по UART;
3. по запросу скрипта — **Press RESET** на модуле в режиме программирования.

**Опции:**

| Команда | Назначение |
|---------|------------|
| `flash_winc.bat 9` | Скачать + прошить |
| `flash_winc.bat 9 --no-fetch` | Прошить без загрузки с интернета |

**Требования:** Windows, Python 3, USB-UART, модуль WINC в режиме программирования, интернет (если не `--no-fetch`).

COM по умолчанию — в `Scripts\config.bat`:

```bat
set "WINC_COM=9"
```

---

## Связь двух способов

| Способ | Когда использовать |
|--------|-------------------|
| `flash_winc.bat` | Первая прошивка, отладка, нет Wi‑Fi |
| `publish_ota_github.bat` + `wifi ota` | Обновление в поле по Wi‑Fi с этого репозитория |

Обычный порядок при выпуске новой версии Microchip:

1. `publish_ota_github.bat` — обновить GitHub;
2. пересобрать iEMAT1 (при смене версии — поправить драйвер);
3. на приборе: `wifi ota <ssid> <password>`.
