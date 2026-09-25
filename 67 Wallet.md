# 📚 Документация: 67 Note Bot

## 📋 Содержание

1. [Обзор](#1-обзор)
2. [Возможности](#2-возможности)
3. [Архитектура](#3-архитектура)
4. [Установка](#4-установка)
5. [Настройка](#5-настройка)
6. [Запуск](#6-запуск)
7. [Команды бота](#7-команды-бота)
8. [Игровая механика](#8-игровая-механика)
9. [Проверки](#9-проверки)
10. [База данных](#10-база-данных)
11. [Конфигурация](#11-конфигурация)
12. [Docker](#12-docker)
13. [Устранение неполадок](#13-устранение-неполадок)
14. [Дальнейшее развитие](#14-дальнейшее-развитие)

---

## 1. Обзор

**67 Note Bot** — Telegram-бот, который начисляет внутреннюю валюту **67 Coin** за запись видеосообщений (кружков) с произнесением числа «67» и показом соответствующего жеста.

**Ключевая идея:** пользователь делает то, что и так делает в TikTok/Reels, но получает за это награду во внутренней валюте экосистемы 67.

**Слоган:** *«Скажи 67 — получи 67»*

---

## 2. Возможности

### Основные

| Функция | Описание |
|---|---|
| Приём кружков | Обработка `video_note` от Telegram |
| Распознавание речи | Whisper определяет «67» в аудио |
| Распознавание жестов | MediaPipe HandLandmarker ищет жест «6» или «7» |
| Проверка лица | InsightFace сверяет лицо в видео с Face ID |
| Начисление монет | До 3 монет за кружок |
| Дневной лимит | 67 кружков в день |
| Бонусы | За 67 монет в день + за ежедневный вход |
| Уровни | 5 уровней держателя |
| Топ игроков | Топ-67 по балансу |

### Защита от накрутки

- Проверка уникальности кружка (`file_unique_id`)
- Ограничение длительности: 5.7–7.7 сек
- Дневной лимит: 67 кружков
- Проверка Face ID — нельзя записать кружок за другого
- Проверка жеста — сложнее подделать автоматически
- Лимит 3 «67» за один кружок

---

## 3. Архитектура

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Telegram    │ ───► │   aiogram    │ ───► │   Router     │
│  Bot API     │      │   Dispatcher │      │   handlers   │
└──────────────┘      └──────────────┘      └──────┬───────┘
                                                    │
                       ┌────────────────────────────┼────────────────────────┐
                       │                            │                        │
                       ▼                            ▼                        ▼
              ┌────────────────┐         ┌────────────────────┐    ┌────────────────┐
              │   Whisper      │         │  MediaPipe Hands   │    │  InsightFace   │
              │  (аудио→текст) │         │     (жесты)        │    │    (лицо)      │
              └────────────────┘         └────────────────────┘    └────────────────┘
                       │                            │                        │
                       └────────────────────────────┼────────────────────────┘
                                                    ▼
                                          ┌────────────────────┐
                                          │   SQLite           │
                                          │   (users, stats)   │
                                          └────────────────────┘
```

### Используемые технологии

| Компонент | Назначение |
|---|---|
| **aiogram 3.x** | Асинхронный фреймворк для Telegram |
| **faster-whisper** | Распознавание речи |
| **MediaPipe Tasks** | Распознавание жестов рук |
| **InsightFace** | Проверка лица по эмбеддингам |
| **OpenCV** | Работа с видео и кадрами |
| **aiosqlite** | Асинхронная работа с SQLite |
| **ffmpeg** | Извлечение аудио из видео |

---

## 4. Установка

### Требования

- **Python 3.11** (рекомендуется), **3.10–3.12** (совместимо)
- **ffmpeg** в PATH
- **RAM ≥ 4 ГБ**
- **CPU ≥ 2 ядра**

> ⚠️ Python 3.13 и 3.14 **не поддерживаются** — библиотеки `mediapipe`, `insightface`, `onnxruntime` не имеют сборок для них.

### Шаг 1: Установи ffmpeg

**Windows:**
```powershell
winget install Gyan.FFmpeg
```

**macOS:**
```bash
brew install ffmpeg
```

**Ubuntu/Debian:**
```bash
sudo apt install ffmpeg
```

Проверь:
```bash
ffmpeg -version
```

### Шаг 2: Клонируй проект

```bash
git clone <repo-url> 67note-bot
cd 67note-bot
```

Или создай папку вручную и положи туда файлы.

### Шаг 3: Создай виртуальное окружение

**Windows:**
```powershell
py -3.11 -m venv venv
venv\Scripts\Activate.ps1
```

**macOS / Linux:**
```bash
python3.11 -m venv venv
source venv/bin/activate
```

### Шаг 4: Установи зависимости

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Шаг 5: Скачай модель жестов

Помести файл `hand_landmarker.task` в корень проекта.

Скачать: https://storage.googleapis.com/mediapipe-models/hand_landmarker/hand_landmarker/float16/1/hand_landmarker.task

### Шаг 6: Проверь импорты

```bash
python -c "import aiosqlite, aiogram, cv2, mediapipe, insightface, faster_whisper; print('OK')"
```

Должно вывести `OK`.

---

## 5. Настройка

Создай файл `.env` в корне проекта:

```env
BOT_TOKEN=8922528058:AAG5TODMCykjLsLcOOGZoNRiHHnfdlCDHUE
WHISPER_MODEL=small

FACE_THRESHOLD=0.35
REQUIRE_FACE_MATCH=true
REQUIRE_GESTURE=true
GESTURE_STRICT=false
```

### Получение токена бота

1. Открой Telegram → найди **@BotFather**
2. Отправь `/newbot`
3. Придумай имя и username
4. Скопируй токен вида `123456789:AA...` в `BOT_TOKEN`

> ⚠️ **Не публикуй токен в открытых репозиториях.** Если утёк — отзови через `/revoke` в BotFather.

---

## 6. Запуск

### Локально

```bash
python bot.py
```

Ожидаемые логи:

```
Загружаю Whisper: small...
Whisper готов.
Загружаю InsightFace (buffalo_l)...
InsightFace готов.
Бот запущен. Polling...
```

**Первый запуск** скачает модели:
- Whisper `small` — ~500 МБ
- InsightFace `buffalo_l` — ~300 МБ

Занимает 5–10 минут в зависимости от скорости интернета.

### Через Docker

```bash
docker compose up -d --build
docker compose logs -f
```

Остановить:

```bash
docker compose down
```

Полный сброс (включая модели):

```bash
docker compose down -v
```

---

## 7. Команды бота

| Команда | Описание |
|---|---|
| `/start` | Регистрация, приветствие, бонус за вход |
| `/register_face` | Инструкция по регистрации Face ID |
| `/balance` (`/bal`) | Баланс, стрик, уровень |
| `/top` | Топ-67 игроков по балансу |

### Отправка фото

Любое обычное фото (не файлом) → бот пытается извлечь лицо и сохранить как Face ID.

### Отправка кружка

Любой `video_note` длительностью 5.7–7.7 сек → обработка и начисление.

---

## 8. Игровая механика

### Начисление монет

| Событие | Награда |
|---|---|
| Одно «67» в кружке | +1 монета |
| До 3 «67» в одном кружке | +3 монеты (максимум) |
| 67 монет за день | +67 монет (бонус) |
| Ежедневный вход | +67 монет |

### Уровни держателя

| Уровень | Баланс | Титул |
|---|---|---|
| 1 | 0–66 | Новичок |
| 2 | 67–669 | Ученик |
| 3 | 670–6 699 | Практик |
| 4 | 6 700–66 999 | Мастер |
| 5 | 67 000+ | Хранитель Баланса |

### Что считается «67»

Распознаются варианты:

- `67` (цифрами)
- `6 7`
- `шестьдесят семь`
- `шесть семь`
- `six seven`
- `sixseven`

### Дневной лимит

- Максимум **67 кружков в день**
- После лимита монеты не начисляются до следующего дня

---

## 9. Проверки

### 9.1 Проверка длительности

- Минимум: **5.7 сек**
- Максимум: **7.7 сек**

Кружки вне диапазона отклоняются.

### 9.2 Проверка уникальности

Каждый кружок имеет `file_unique_id`. Повторная отправка того же кружка → 0 монет.

### 9.3 Проверка лица (Face ID)

**Как работает:**
1. Пользователь отправляет селфи → сохраняется embedding (вектор лица)
2. При обработке кружка извлекаются кадры
3. InsightFace находит лица на кадрах
4. Считается косинусная схожесть с сохранённым embedding
5. Если схожесть ≥ `FACE_THRESHOLD` → проверка пройдена

**Порог:**
- `0.35` — мягко (по умолчанию)
- `0.50` — строго

**Отключить:** `REQUIRE_FACE_MATCH=false`

### 9.4 Проверка жеста

**Как работает:**
1. MediaPipe HandLandmarker находит руки на кадрах
2. Определяет, какие пальцы подняты
3. Проверяет паттерн:
   - **«6»** — большой + мизинец
   - **«7»** — большой + указательный + средний

**Режимы:**
- `GESTURE_STRICT=false` — достаточно любой руки с поднятыми пальцами
- `GESTURE_STRICT=true` — только паттерн «6» или «7»

**Отключить:** `REQUIRE_GESTURE=false`

### 9.5 Распознавание речи

- **Модель:** faster-whisper
- **Опции:** `vad_filter=True`, `beam_size=1`
- **Языки:** автоопределение

**Размеры моделей:**

| Модель | Размер | Скорость | Точность |
|---|---|---|---|
| `tiny` | ~40 МБ | очень быстро | низкая |
| `base` | ~150 МБ | быстро | средняя |
| `small` | ~500 МБ | средне | хорошая |
| `medium` | ~1.5 ГБ | медленно | высокая |

---

## 10. База данных

**Файл:** `data/users.db` (SQLite)

### Таблица `users`

| Поле | Тип | Описание |
|---|---|---|
| `user_id` | INTEGER PK | Telegram ID |
| `username` | TEXT | @username |
| `balance` | INTEGER | Баланс 67 Coin |
| `total_67` | INTEGER | Всего произнесено «67» |
| `streak` | INTEGER | Дней подряд |
| `last_login` | TEXT | Дата последнего входа |
| `face_embedding` | BLOB | Вектор лица |
| `created_at` | TEXT | Дата регистрации |

### Таблица `daily_stats`

| Поле | Тип | Описание |
|---|---|---|
| `user_id` | INTEGER | Telegram ID |
| `day` | TEXT | Дата (ISO) |
| `videos_count` | INTEGER | Кружков за день |
| `coins_earned` | INTEGER | Монет за день |
| `bonus_67` | INTEGER | Получен ли бонус за 67 монет |

PK: `(user_id, day)`

### Таблица `seen_videos`

| Поле | Тип | Описание |
|---|---|---|
| `file_unique_id` | TEXT PK | Уникальный ID кружка |
| `user_id` | INTEGER | Кто отправил |
| `created_at` | TEXT | Когда |

### Резервное копирование

```bash
# Скопируй файл
cp data/users.db backup/users_$(date +%Y%m%d).db
```

Для Docker:

```bash
docker cp 67note-bot:/app/data/users.db ./backup/
```

---

## 11. Конфигурация

Все параметры задаются через **переменные окружения** или **`.env`**.

### Основные

| Переменная | По умолчанию | Описание |
|---|---|---|
| `BOT_TOKEN` | — | Токен Telegram-бота |
| `WHISPER_MODEL` | `small` | Размер модели Whisper |

### Проверки

| Переменная | По умолчанию | Описание |
|---|---|---|
| `FACE_THRESHOLD` | `0.35` | Порог схожести лиц |
| `REQUIRE_FACE_MATCH` | `true` | Требовать Face ID |
| `REQUIRE_GESTURE` | `true` | Требовать жест |
| `GESTURE_STRICT` | `false` | Только «6»/«7» |

### Игровые (в коде `bot.py`)

```python
COINS_PER_67 = 1          # монет за одно «67»
MAX_PER_VIDEO = 3         # максимум «67» за кружок
DAILY_LIMIT = 67          # кружков в день
MIN_DURATION = 5.7        # мин. длина (сек)
MAX_DURATION = 7.7        # макс. длина (сек)
BONUS_67_PER_DAY = 67     # бонус за 67 монет в день
DAILY_LOGIN_BONUS = 67    # бонус за вход
```

---

## 12. Docker

### Файлы

- `Dockerfile` — образ с Python 3.11, ffmpeg, зависимостями
- `docker-compose.yml` — оркестрация + volume для моделей

### Команды

```bash
# Собрать и запустить
docker compose up -d --build

# Логи
docker compose logs -f

# Перезапустить
docker compose restart

# Остановить
docker compose down

# Остановить + удалить модели и БД
docker compose down -v
```

### Volume

| Volume | Назначение |
|---|---|
| `./data` | База данных SQLite |
| `insightface_models` | Модели InsightFace |
| `whisper_cache` | Модели Whisper |

### Ресурсы

```yaml
deploy:
  resources:
    limits:
      memory: 4g
    reservations:
      memory: 1g
```

Если RAM меньше — уменьши `WHISPER_MODEL` до `tiny` или `base`.

---

## 13. Устранение неполадок

### ❌ `ModuleNotFoundError: No module named 'aiosqlite'`

**Причина:** библиотеки не установлены или установлены не в тот Python.

**Решение:**
```powershell
python -m pip install -r requirements.txt
python -c "import aiosqlite; print('OK')"
```

Если `python -m pip` не работает — проверь, что venv активирован:
```powershell
python -c "import sys; print(sys.executable)"
pip -V
```
Пути должны совпадать.

---

### ❌ `No matching distribution found for onnxruntime==1.18.0`

**Причина:** Python 3.13+ не поддерживается.

**Решение:** установи Python 3.11 и пересоздай venv:
```powershell
Remove-Item -Recurse -Force venv
py -3.11 -m venv venv
venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

---

### ❌ `Import "mediapipe.python.solutions" could not be resolved`

**Причина:** в новых версиях MediaPipe (0.10.30+) удалён старый API `solutions`.

**Решение 1 (быстрое):** откатить MediaPipe:
```powershell
python -m pip uninstall mediapipe -y
python -m pip install mediapipe==0.10.21
```

**Решение 2 (правильное):** переписать блок жестов на `mediapipe.tasks`.

---

### ❌ `OSError: [WinError 121] Превышен таймаут семафора`

**Причина:** нет доступа к `api.telegram.org` (блокировка провайдером, файрвол, антивирус).

**Решение:**
1. Проверь в браузере: `https://api.telegram.org`
2. Если не открывается — включи VPN
3. Если открывается — добавь `python.exe` в исключения брандмауэра
4. Или используй прокси в коде:

```python
from aiogram.client.session.aiohttp import AiohttpSession

session = AiohttpSession(proxy="socks5://user:pass@host:port")
bot = Bot(token=BOT_TOKEN, session=session, ...)
```

---

### ❌ `ffmpeg not found`

**Причина:** ffmpeg не установлен или не в PATH.

**Решение:**

Windows:
```powershell
winget install Gyan.FFmpeg
```
Перезапусти терминал.

Проверь:
```powershell
ffmpeg -version
```

---

### ❌ Бот не реагирует на кружки

**Возможные причины:**
- Длительность вне диапазона 5.7–7.7 сек
- Face ID не зарегистрирован (`REQUIRE_FACE_MATCH=true`)
- Жест не распознан (`REQUIRE_GESTURE=true`)
- Дневной лимит исчерпан (67 кружков)
- Кружок уже был засчитан

**Проверь логи** — там будет причина отклонения.

---

### ❌ Медленная обработка

**Причина:** Whisper + InsightFace на CPU тяжёлые.

**Решения:**
1. Уменьши модель: `WHISPER_MODEL=tiny`
2. Отключи Face ID: `REQUIRE_FACE_MATCH=false`
3. Отключи жест: `REQUIRE_GESTURE=false`
4. Используй GPU (см. раздел «Дальнейшее развитие»)

---

## 14. Дальнейшее развитие

### Готовые направления

| Направление | Что даёт |
|---|---|
| **PostgreSQL** | Замена SQLite для нагрузки |
| **Redis + очередь** | Асинхронная обработка видео |
| **GPU-режим** | Ускорение в 5–10× |
| **Webhook** | Вместо polling для продакшена |
| **SPL-токен Solana** | Вывод 67 Coin в кошелёк |
| **Админ-панель** | Модерация, статистика |
| **Мультиязычность** | Английский, другие языки |

### GPU-режим

Замени в `requirements.txt`:
```
onnxruntime-gpu==1.18.0
```

В `bot.py`:
```python
_face_app = FaceAnalysis(name="buffalo_l", providers=["CUDAExecutionProvider"])
```

В `docker-compose.yml`:
```yaml
deploy:
  resources:
    reservations:
      devices:
        - driver: nvidia
          count: 1
          capabilities: [gpu]
```

Ускорение: Whisper ~10×, InsightFace ~5×.

### Webhook вместо polling

```python
from aiogram.webhook.aiohttp_server import SimpleRequestHandler, setup_application
from aiohttp import web

WEBHOOK_URL = "https://your-domain.com/webhook"
WEBHOOK_PATH = "/webhook"

async def on_startup(bot):
    await bot.set_webhook(f"{WEBHOOK_URL}{WEBHOOK_PATH}")

app = web.Application()
SimpleRequestHandler(dispatcher=dp, bot=bot).register(app, path=WEBHOOK_PATH)
setup_application(app, dp, bot=bot)
web.run_app(app, host="0.0.0.0", port=8080)
```

### Интеграция 67 Coin с Solana

1. Создай SPL-токен:
   ```bash
   spl-token create-token
   spl-token create-account <MINT>
   spl-token mint <MINT> 6700000000
   ```

2. Добавь в `bot.py` endpoint вывода:
   ```python
   from solana.rpc.api import Client
   from spl.token.client import Token

   async def withdraw(user_id: int, wallet: str):
       # проверка баланса, лимитов, KYC
       # отправка токена через SPL
       ...
   ```

3. Добавь команду `/withdraw <wallet>`.

> ⚠️ Требует юридической обвязки: KYC/AML, лицензия, если торгуется на бирже.

---

## 📎 Приложение: структура проекта

```
67note-bot/
├── bot.py                  # основной код
├── requirements.txt        # зависимости
├── .env                    # токен и настройки (не коммитить!)
├── Dockerfile              # образ Docker
├── docker-compose.yml      # оркестрация
├── hand_landmarker.task    # модель жестов
├── DOCUMENTATION.md        # этот файл
└── data/
    └── users.db            # БД (создаётся автоматически)
```

---

## 📞 Контакты и поддержка

- **BotFather:** https://t.me/BotFather
- **aiogram:** https://docs.aiogram.dev
- **faster-whisper:** https://github.com/SYSTRAN/faster-whisper
- **MediaPipe:** https://ai.google.dev/edge/mediapipe
- **InsightFace:** https://github.com/deepinsight/insightface

---

**Версия документации:** 1.0  
**Совместимость:** 67 Note Bot v1.x  
**Дата:** 2026
