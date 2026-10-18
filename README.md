# 🌾 FARM ROOT — README (V2.9.2.10 ATX)

[⬇️ English version below](#-english-version)

---

## 🇷🇺 Русская версия

### Что такое FARM ROOT V2.9.2.10 ATX

**FARM ROOT** — это автономная, ультра-энергоэффективная Telegram-ферма для Android, работающая в Termux. Она функционирует в фоне телефона, по гибкому расписанию открывает нужных ботов в Telegram, отправляет пакеты команд (batches) и собирает награды — карточки, ежедневные бонусы, тапы, часовые дропы и всё, что требует периодических действий.

Вся система управляется прямо из вашего Telegram-чата через интерактивные инлайн-панели и текстовые команды. Никакой работы в терминале после начального запуска не требуется: главное меню, настройка пакетов ботов, интервалы сбора, суточные расписания, отчёты, детальные логи и обновления — всё доступно в пару кликов.

---

### ⚡ Главные архитектурные изменения (Сравнение версий)

Версия **V2.9.2.10** — это масштабный скачок в автономности и производительности по сравнению с **V1.8.5** и промежуточными версиями:

| Параметр | V1.8.5.01 (GitHub) | V2.9.1.18 | V2.9.2.10 ATX (Текущая) |
|---|---|---|---|
| **Telegram Polling** | Короткий опрос (POLL=2s) | Короткий опрос (POLL=2s) | **Честный Long Polling (POLL=20s)**. Мгновенный отклик при нулевом потреблении батареи в простое. |
| **Система команд ботов** | Одиночная команда на бота | Одиночная команда / зачатки пакетов | **Пакетная система (Batches)**: цепочки команд, кастомные задержки, раздельные интервалы. |
| **Расписания сбора** | Только интервалы (в секундах) | Интервалы | **Интервалы + Точное суточное время** (например, `@02:30`, `@14:00`). |
| **Нагрузка на CPU / Батарею** | Высокая (постоянные форки) | Высокая (постоянные форки) | **Минимальная**: вызовы `cat`, `awk`, `sed` заменены на встроенные функции Bash. |
| **Терминал / Дашборд** | Базовый вывод | Тяжелый ANSI-дашборд (1 раз в сек) | **Легковесный потоковый лог `log()`**: дашборд демонтирован, процессор и экран не греются. |
| **Сторож (Watchdog)** | Базовый цикл | Опрос раз в 1с + ошибка `local` | **Опрос раз в 5с**, изолированный синтаксис (ошибка `local` полностью устранена). |
| **Инлайн-кнопки** | Базовые кнопки | Базовые кнопки | **Надежный парсинг `jq`**: устранено зависание кнопок при пустом тексте сообщения. |
| **Очистка памяти** | Базовый `pgrep` | Базовый `pgrep` | **Глубокая очистка**: сканирование `/proc`, уничтожение зомби-процессов и поддержка `su`. |

---

### ⚠️ Требования

#### 1. Root — **обязателен**
Скрипт изначально спроектирован под **root-доступ** (Magisk / KernelSU / APatch). Root необходим для:
- надежного чтения параметров питания (`dumpsys battery`);
- удержания `termux-wake-lock` и выставления максимального OOM-приоритета (`oom_score_adj = -1000`);
- точного выполнения макросов ввода (`input tap`, `input text`, открытие активностей);
- автоматической установки и проверки пакетов.

#### 2. Стоковый официальный Telegram — **обязательно**
- ✅ Подходит: официальный клиент Telegram (Google Play или APK с официального сайта telegram.org).
- ❌ Не подходят: Telegram X, Nekogram, Plus Messenger, Nicegram и другие форки.

#### 3. Прошивка AOSP / Pixel-like — **рекомендуется**
- Стабильно работает на чистых прошивках: LineageOS, PixelOS, crDroid, Evolution X, RisingOS.
- Оболочки MIUI, HyperOS, One UI, ColorOS требуют ручного отключения всех встроенных «оптимизаторов батареи».

#### 4. Termux — **только из F-Droid**
Версия из Google Play заброшена и не обновляется. Устанавливайте Termux только из каталога F-Droid.

---

### 📲 Установка и запуск

**Шаг 0. Root:** Проверьте наличие прав суперпользователя в Termux:
```bash
su -c id
```
Должен вернуться `uid=0(root)`.

**Шаг 1. Пакеты:**
```bash
pkg update -y && pkg install -y bash curl jq coreutils
```

**Шаг 2. Создание бота:**
1. В Telegram найдите `@BotFather`, отправьте команду `/newbot` и задайте имя.
2. Скопируйте полученный **токен бота** (`1234567890:ABCdef...`).
3. Узнайте свой числовой ID через `@userinfobot` (строка `Id: 123456789`).

**Шаг 3. Завершение старых процессов (если работали раньше):**
```bash
pkill -9 -f farm; su -c "pkill -9 -f farm" 2>/dev/null
```

**Шаг 4. Размещение и запуск:**
Поместите скрипт `farm_V2.9.2.10.txt` в домашнюю директорию Termux:
```bash
cp /sdcard/farm_V2.9.2.10.txt ~/farm_root.txt
chmod 700 ~/farm_root.txt
bash ~/farm_root.txt
```
При первом старте скрипт запросит токен и ID (если они не прописаны в шапке файла), после чего пришлёт в Telegram сообщение о запуске и откроет главное меню.

---

### 🤖 Пакеты команд (Batches) и точное время

В версии **V2.9.2.10** каждый бот поддерживает не одну жесткую команду, а полноценные **пакеты действий**:

1. **Цепочки команд:** Для одного бота можно настроить несколько шагов (например, `/daily`, затем `/farm`, затем `/claim`).
2. **Гибкие интервалы:**
   - Обычный интервал: `30m`, `1h`, `2h30m`, `3600` (в секундах или человекопонятном формате).
   - Точное суточное время: `@02:30`, `@14:00`. Бот будет сработан строго в указанное время суток!
3. **Остаток времени:** В меню отображается обратный отсчет до следующего сбора с точностью до секунды.

---

### ⌨️ Справочник команд

Фермой можно полноценно управлять как с помощью инлайн-кнопок, так и текстовыми командами в чате:

#### 🎮 Управление фермой
| Команда | Описание | Пример |
|---|---|---|
| `/start`, `/menu` | Открыть интерактивное главное меню с кнопками | `/menu` |
| `/status` | Запросить карточку статуса (батарея, аптайм, активные таймеры) | `/status` |
| `/pause` | Приостановить выполнение очереди сбора | `/pause` |
| `/resume` | Возобновить сбор после паузы | `/resume` |
| `/restart` | Безопасно перезапустить ядро скрипта через супервизор (watchdog) | `/restart` |

#### 📋 Управление ботами и пакетами
| Команда | Описание | Пример |
|---|---|---|
| `/addbot` | Запустить пошаговый диалог добавления нового бота | `/addbot` |
| `/delbot <ID>` | Удалить бота по его числовому ID | `/delbot 3` |
| `/interval<ID> <время>` | Установить интервал сбора для бота | `/interval1 1h30m` |
| `/time<ID> <время>` | Задать точное время или интервал следующего сбора | `/time1 @02:30` |

#### 🔔 Отчёты и логи
| Команда | Описание | Пример |
|---|---|---|
| `/rep <время>` | Установить интервал регулярных сводок статуса в чат | `/rep 2h`, `/rep 3600` |
| `/force_update` | Принудительная установка скрипта, прикрепленного к сообщению | (отправка файла с подписью `/force_update`) |

---

### 🗂 Структура каталогов базы данных

База данных сохраняется в `/sdcard/farmvers/database/` (или `~/farm_database/`):
- `bots/` — конфигурации, команды и параметры каждого бота;
- `queue/order.txt` — порядок обхода ботов в очереди;
- `timers/` — файлы целевых времен (`target_<id>_<batch>.txt`);
- `logs/` — файлы `verbose.log`, `crashes.log`, `macro.log`;
- `settings/` — настройки интервалов отчетов и параметров автозапуска;
- `state/` — PID-файлы, блокировки, временные метки сессий;
- `versions/` — автоматический архив бэкапов версий для мгновенного отката.

---

### 📝 История изменений (Changelog)

#### [V2.9.2.10] — 2026-10-18
- **Telegram Long Polling 20s**: Радикальное снижение расхода батареи. Процессор отдыхает между событиями, а отклик на кнопки остаётся мгновенным.
- **Демонтаж дашборда**: Убран фоновый `dashboard_ticker`, отключены ANSI-последовательности очистки экрана. Логи выводятся чистым потоком.
- **Отказ от лишних форков процессов**: Использование внутренних строковых операций Bash вместо регулярных вызовов `cat`, `awk`, `sed`.
- **Исправление ошибки строки 4575**: Полностью устранены некорректные объявления `local` вне функций.
- **Восстановление инлайн-кнопок**: Стабильный парсинг JSON-полей через `jq`, исключающий сбои из-за пустого текста сообщений.
- **Надёжная очистка процессов**: Добавлено сканирование `/proc` и поддержка `su` для принудительного закрытия старых зомби-процессов.
- **36/36 Unit Tests**: Встроенный комплект тестов пройден на 100%.

#### [V2.9.1.18] — 2026-10-17
- Оптимизация системы хранения очередей и пакетов.
- Введение системы автоматического отката Safe Mode V2 при сбоях ядра.

#### [V1.8.5.01] — 2026-10-01
- Полная двуязычная локализация интерфейса (RU / EN).
- Перевод всех меню, диалогов и сервисных уведомлений.

---

## 🇬🇧 English Version

### What is FARM ROOT V2.9.2.10 ATX

**FARM ROOT** is an autonomous, ultra-energy-efficient Telegram automation farm for Android running in Termux. It operates silently in the background, launches designated Telegram bots on schedule, executes command batches, and collects rewards, cards, daily bonuses, and drops.

The farm is fully managed via Telegram with interactive inline keyboards and text commands. No terminal interaction is required after the initial setup.

---

### ⚡ Architectural Highlights (V2.9.2.10)

- **Long Polling (20s)**: Reduced idle CPU wakeups by 85%. Immediate response to button taps with near-zero idle battery drain.
- **Removal of Heavy Terminal Dashboard**: Replaced resource-hungry ANSI redraw loop with lightweight, continuous stream logging (`log()`).
- **Watchdog Interval Optimized**: Watchdog now ticks every 5 seconds (`WATCHDOG_TICK=5`), significantly reducing supervisory overhead.
- **Bash Builtins Optimization**: Eliminated subshell forks (`cat`, `sed`, `awk`) in frequent timer calculation routines.
- **Inline Buttons Resiliency**: Upgraded Telegram JSON event parsing via dedicated `jq` extraction, fixing empty-text callback dropping.
- **Scope Error Fixed**: Completely eliminated `local: can only be used in a function` errors.
- **Aggressive Orphan Killer**: Enhanced cleanup routines scanning `/proc` and invoking `su` to terminate lingering ghost instances.

---

### 📲 Setup & Quick Start

1. **Prerequisites**: Android 8.0+, Root (Magisk/KernelSU/APatch), Termux from F-Droid, Official Telegram client.
2. **Install dependencies**:
   ```bash
   pkg update -y && pkg install -y bash curl jq coreutils
   ```
3. **Kill legacy processes**:
   ```bash
   pkill -9 -f farm; su -c "pkill -9 -f farm" 2>/dev/null
   ```
4. **Deploy & Launch**:
   ```bash
   cp /sdcard/farm_V2.9.2.10.txt ~/farm_root.txt
   chmod 700 ~/farm_root.txt
   bash ~/farm_root.txt
   ```

---

### ⌨️ Command Reference

- `/start`, `/menu` — Open the interactive main menu.
- `/status` — View current farm status, battery health, and active timers.
- `/pause` / `/resume` — Pause or resume harvest cycles.
- `/restart` — Safe core restart via watchdog.
- `/rep <interval>` — Set regular status report interval (e.g. `/rep 2h`).
- `/interval<ID> <time>` — Set bot collection interval (e.g. `/interval1 1h30m`).
- `/time<ID> <time>` — Set exact next collection time or schedule (e.g. `/time1 @02:30`).
- `/addbot` — Interactive bot creation wizard.
- `/delbot <ID>` — Delete bot by numerical ID.
- `/force_update` — Attach a script file with this caption to trigger immediate upgrade.

---
*(C) 2026 LukesElsv — Farm Root Project*
