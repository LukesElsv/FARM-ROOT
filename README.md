# 🌾 FARM ROOT — Руководство и Документация (V2.9.2.10 ATX)

[⬇️ English version below](#-english-version)

---

## 🇷🇺 Русская версия

### Что такое FARM ROOT V2.9.2.10 ATX

**FARM ROOT** — это автономная, ультра-энергоэффективная система автоматизации и сбора для Telegram-ботов на Android, работающая в окружении **Termux**. Скрипт функционирует в фоновом режиме, удерживает OOM-приоритет, по гибким расписаниям открывает целевых ботов в Telegram, отправляет заданные цепочки команд и пакетов (batches) и собирает награды (ежедневные бонусы, карточки, тапы, периодический сбор).

Управление фермой происходит полностью через Telegram — с помощью интерактивных инлайн-панелей и текстовых команд. После первого запуска открывать терминал не требуется: управление ботами, создание батчей, настройка расписаний, отчёты, логи, резервное копирование и обновления доступны прямо в чате с вашим ботом.

---

### 📦 Система команд и батчей (Batches) — Полное руководство

В архитектуре FARM ROOT каждый бот может содержать **неограниченное количество независимых пакетов команд (батчей)**. Каждый батч имеет собственный таймер, собственный тип запуска, собственное количество повторов и команду.

#### 1. Структура параметров батча:
Каждый батч состоит из 5 ключевых параметров:
1. **Идентификатор батча (`b1`, `b2`, `b3`...)** — уникальный номер задачи внутри бота.
2. **Тип расписания и время:**
   - **Интервал (`interval`)**: повторение через заданный промежуток времени. Поддерживаются форматы:
     - `02:30` или `0230` — 2 часа 30 минут;
     - `90m` — 90 минут;
     - `2h` — 2 часа;
     - `5400` — 5400 секунд.
   - **Точное суточное время (`time`)**: запуск строго в назначенное время суток каждый день:
     - `@02:30` — ежедневно в 02:30 ночи;
     - `@14:00` — ежедневно в 14:00 дня;
     - `@23:59` — ежедневно без одной минуты полночь.
3. **Количество повторов за запуск (`xN`):**
   - `x1` (по умолчанию) — отправить команду один раз;
   - `x2`, `x5`, `x10` — отправить команду N раз подряд с паузой `BATCH_REPEAT_DELAY` (по умолчанию 1 сек). Удобно для тапов или многократного сбора.
4. **Счётчик оставшихся выполнений (`rN` — remaining runs):**
   - `r0` (по умолчанию) — бессрочный батч (работает бесконечно);
   - `r1`, `r5`, `r10` — временный/акционный батч. После N успешных срабатываний батч **автоматически удаляется** из конфигурации бота.
5. **Целевая команда (`cmd`):**
   - Любая команда для бота: `/claim`, `/daily`, `/farm`, `/bonus`, `/start` и т.д.

---

#### 2. Добавление бота и батчей одной строкой (`/addbot`)

Формат однострочной команды:
```text
/addbot @username [1|0] [Эмодзи] Имя. <спецификация_батчей>
```
* `1|0`: разрешить (1) или запретить (0) быстрое изменение интервала через меню.
* Точка `.` **строго разделяет** имя бота и параметры батчей!

##### Примеры создания:
1. **Простой бот с одной командой и дефолтным интервалом (1 час):**
   ```text
   /addbot @coolbot 1 🐥 Цыплята. /claim
   ```
2. **Бот с интервалом 2 часа 30 минут и двумя командами подряд:**
   ```text
   /addbot @coolbot 1 🐥 Цыплята. 02:30 /claim /crack
   ```
   *(Создаст 2 батча: `b1` с интервалом 2:30 и командой `/claim`, `b2` с интервалом 2:30 и командой `/crack`)*
3. **Бот с разными независимыми батчами:**
   ```text
   /addbot @coolbot 1 🐥 Цыплята. b1 01:00 /daily b2 03:00 x5 /tap
   ```
   *(`b1` собирает `/daily` каждый час; `b2` раз в 3 часа нажимает `/tap` 5 раз подряд)*
4. **Бот с запуском по точному суточному времени:**
   ```text
   /addbot @coolbot 1 🐥 Цыплята. @14:30 /daily
   ```
   *(Срабатывает ровно в 14:30 каждый день)*
5. **Акционный батч с лимитом срабатываний (исчезнет через 5 сборов):**
   ```text
   /addbot @coolbot 1 🐥 Цыплята. 02:00 r5 /promo
   ```

---

#### 3. Интерактивный мастер управления батчами через Telegram

В меню любого бота доступен раздел **«📝 Команды / Батчи»**:
* Просмотр всех настроенных батчей бота с их таймерами, интервалами и повторами.
* **Инлайн-редактирование каждого батча:**
  - `✏️ Сменить команду` — ввод новой команды сообщением;
  - `⏱ Сменить время` — переключение между интервалом (`1h30m`) и точным временем (`@02:30`);
  - `🔁 Повторы xN` — настройка количества отправок (`x1`–`x99`);
  - `🔄 Лимит rN` — установка срока жизни (`r0` — бессрочно, `rN` — N раз);
  - `🗑 Удалить батч` — удаление отдельного шага.
* **«➕ Добавить команду»** — пошаговый мастер добавления нового батча прямо в диалоге с ботом без ввода сложных строк.

---

### ⌨️ Полный справочник текстовых команд

| Команда | Описание | Пример использования |
|---|---|---|
| `/menu`, `/start` | Открыть главное меню с инлайн-кнопками | `/menu` |
| `/status` | Вывести текущую сводку: батарея, статус очереди, активные таймеры | `/status` |
| `/farm` | Принудительно запустить полный цикл сбора по всем активным ботам сейчас | `/farm` |
| `/farm<ID>` | Запустить сбор только одного конкретного бота вне очереди | `/farm1`, `/farm2` |
| `/pause` | Поставить сбор на паузу (очередь замораживается) | `/pause` |
| `/start_farm`, `/resume` | Снять с паузы и продолжить работу | `/resume` |
| `/bots` | Открыть интерактивный список всех добавленных ботов | `/bots` |
| `/queue` | Открыть управление очередью и порядком ботов | `/queue` |
| `/run` | Панель ручного запуска задач | `/run` |
| `/rep <время>` | Задать периодичность отправки регулярных отчётов фермы | `/rep 2h`, `/rep 3600` |
| `/interval<ID> <время>` | Быстро изменить интервал бота по его номеру | `/interval1 1h30m` |
| `/time<ID> <время>` | Отложить следующий сбор бота на заданное время или назначить час | `/time1 45m`, `/time1 @18:00` |
| `/delbot<ID>` | Удалить бота по его числовому номеру | `/delbot2` |
| `/botname<ID> <имя>` | Сменить отображаемое имя бота | `/botname1 💎 Алмазы` |
| `/botcmd<ID> <команда>` | Сменить команду первого батча бота | `/botcmd1 /daily` |
| `/botuser<ID> <username>` | Сменить Telegram username бота | `/botuser1 @new_bot` |
| `/boticon<ID> <эмодзи>` | Сменить иконку-эмодзи бота | `/boticon1 🚀` |
| `/addbot` | Запустить мастер добавления нового бота | `/addbot` |
| `/macrolog` | Журнал макрокоманд (открытие приложений, ввод, статус кликов) | `/macrolog` |
| `/logs` | Подробный системный лог работы демона | `/logs` |
| `/errhelp` | Интерактивный справочник всех кодов ошибок фермы | `/errhelp` |
| `/update` | Меню проверки и наката обновлений | `/update` |
| `/versions` | Список сохранённых локальных версий для быстрого отката | `/versions` |
| `/saveversion` | Создать именованную резервную копию текущего состояния | `/saveversion` |
| `/language` | Переключение языка интерфейса (RU / EN) | `/language` |
| `/autostart` | Меню автозапуска скрипта при перезагрузке устройства | `/autostart` |
| `/restart` | Безопасный перезапуск ядра через сторожевой процесс (watchdog) | `/restart` |
| `/exp` | Экспорт текущей рабочей версии скрипта в виде `.txt` в чат | `/exp` |
| `/force_update` | Установить прикреплённый скрипт с автоматическим бэкапом | (файл с подписью `/force_update`) |
| `/cancel` | Сбросить текущий пошаговый диалог ввода | `/cancel` |

---

### ⚡ Что изменилось в V2.9.2.10 ATX: Детальное сравнение

#### Сравнение эволюции проекта:

| Параметр | V1.8.5.01BR (Старый релиз GitHub) | V2.9.1.18 (Предыдущая рабочая ветка) | V2.9.2.10 ATX (Актуальный релиз) |
|---|---|---|---|
| **Сетевой опрос (Telegram)** | Короткий опрос `POLL_WAIT=2` (запрос каждые 2 сек) | Короткий опрос `POLL_WAIT=2` (запрос каждые 2 сек) | **Энергосберегающий Long Polling `POLL_WAIT=20`** (до 20 сек). Процессор смартфона спит в простое; отклик на кнопки мгновенный. |
| **Система команд** | Одиночная команда на бота | Поддержка батчей, но с частыми сбоями | **Полнофункциональный движок пакетов (Batches)**: `@HH:MM`, `xN`, `rN`, мастер `addcmd`, парсер однострочников. |
| **Интерфейс консоли** | Базовый вывод в терминал | Тяжелый дашборд с ежесекундным тикером `dashboard_ticker` и ANSI-перерисовкой `[2J[H` | **Полный демонтаж дашборда**: тикер отключен, ANSI-коды удалены. Логи идут чистым, лёгким потоком `log()`, не нагревая дисплей и процессор. |
| **Сторож (Watchdog)** | Базовый супервизор | Опрос каждую 1 сек + ошибка `line 4575: local: can only be used in a function` | **Опрос раз в 5 сек (`WATCHDOG_TICK=5`)**, чистый POSIX-синтаксис без нелегальных `local`. Ошибка ликвидирована. |
| **Инлайн-кнопки меню** | Базовый разбор | Разбор через TSV/read (ломал кнопки при пустом тексте сообщения) | **Индивидуальный парсинг через `jq`**: 100% стабильность инлайн-кнопок и диалогов меню. |
| **Системные вызовы** | Регулярные форки `cat`, `sed`, `awk` | Форки `cat`, `sed`, `awk` в цикле таймеров | **Встроенные операции Bash**: строковые подстановки `${var//...}`, `printf -v`, `read -r` вместо тысяч процессов `fork()`. |
| **Очистка процессов** | Простой `pgrep` | Простой `pgrep` | **Глубокий Kill-Switch**: сканирование `/proc/*/cmdline`, закрытие по PID и вызов `su -c "pkill -9 -f farm"`. |
| **Зависимости** | Требовалась ручная установка `curl`, `jq` | Ручная установка | **Автоматическая установка**: скрипт сам находит и ставит `curl` и `jq` через `pkg`/`apt` при старте. |

---

### ⚠️ Системные требования

1. **Root-доступ (Magisk / KernelSU / APatch)** — обязателен для чтения параметров питания (`dumpsys battery`), удержания OOM-защиты (-1000) и автоматического ввода (`input tap`, `input text`).
2. **Официальный клиент Telegram** — сторонние клиенты (Plus, Nekogram, TG X) не поддерживают стандартные Intent-ссылки передачи параметров.
3. **Termux из F-Droid** — версия из Play Store не поддерживается из-за устаревших репозиториев.
4. **Android 8.0+**.

---

### 📲 Установка и запуск в 3 шага

1. **Остановите все старые процессы фермы в Termux:**
   ```bash
   pkill -9 -f farm; su -c "pkill -9 -f farm" 2>/dev/null
   ```
2. **Разместите файл скрипта `farm_V2.9.2.10.txt`:**
   ```bash
   cp /sdcard/farm_V2.9.2.10.txt ~/farm_root.txt
   chmod 700 ~/farm_root.txt
   ```
3. **Запустите ферму:**
   ```bash
   bash ~/farm_root.txt
   ```
   При первом запуске скрипт запросит токен бота и ваш Chat ID, проверит окружение и сразу пришлёт сообщение об успешном старте.

---

### 🗂 Структура каталогов базы данных

Данные хранятся в `/sdcard/farmvers/database/` (или `~/farm_database/`):
- `bots/<id>/` — файлы бота (`name.txt`, `domain.txt`, `icon.txt`, `batches.txt`);
- `queue/order.txt` — порядок обработки ботов в очереди;
- `timers/` — файлы целевых меток времени запусков пакетов (`target_<id>_<batch>.txt`);
- `logs/` — журналы: `verbose.log`, `crashes.log`, `macro.log`, `macro_report.log`;
- `settings/` — настройки отчётов, автозапуска и расписаний;
- `state/` — PID-файлы, временные маркеры сессий, кеш батареи;
- `versions/` — архив автоматических копий версий для мгновенного отката.

---

### 📝 История изменений (Changelog)

#### [V2.9.2.10] — 2026-10-18
- **Telegram Long Polling 20s**: Процессор переведён в режим глубокого сна в периоды простоя, сетевые запросы сокращены на 85%.
- **Демонтаж дашборда**: Удалён фоновый тикер `dashboard_ticker`, отключены управляющие ANSI-последовательности экрана, логи идут чистым потоком.
- **Исправление синтаксической ошибки**: Устранена ошибка `local: can only be used in a function` на строке 4575 и сопутствующих участках.
- **Восстановление инлайн-кнопок**: Стабильное извлечение JSON-полей через `jq`, исключающее пропуск callback-событий.
- **Оптимизация производительности**: Замена внешних вызовов `cat`, `awk`, `sed` в ядре на встроенные операции Bash.
- **Надёжная очистка памяти**: Интеграция глубокого сканирования `/proc` и поддержка `su` для принудительного закрытия фоновых root-зомби.
- **100% прохождение тестов**: Все 36 из 36 юнит-тестов пройдены успешно.

#### [V2.9.1.18] — 2026-10-17
- Интеграция пакетной архитектуры (Batches) и поддержка расписаний по времени суток (`@HH:MM`).
- Реализация системы автоматического восстановления Safe Mode V2 при аварийных перезапусках.

#### [V1.8.5.01] — 2026-10-01
- Полная двуязычная локализация интерфейса (RU / EN).
- Унификация текстовых слоёв и сервисных подсказок.

---

## 🇬🇧 English Version

### What is FARM ROOT V2.9.2.10 ATX

**FARM ROOT** is an autonomous, ultra-energy-efficient Telegram automation farm for Android running in **Termux**. It operates silently in the background, keeps OOM immunity, opens designated Telegram bots according to customizable schedules, sends multi-step command batches, and collects daily rewards, bonus cards, and drops.

The farm is controlled entirely via Telegram using interactive inline keyboards and text commands. No terminal interactions are needed after the initial launch.

---

### 📦 Command & Batch System (Batches) — Full Guide

In FARM ROOT, each bot can have **multiple independent command batches**. Each batch has its own timer, trigger schedule, repetition count, and target command.

#### Batch Parameters:
1. **Batch ID (`b1`, `b2`, `b3`...)** — Unique task identifier within the bot.
2. **Schedule Type & Time:**
   - **Interval**: periodic repetition (`02:30`, `90m`, `2h`, `5400`).
   - **Exact Daily Time**: triggers every day at a specific time (`@02:30`, `@14:00`).
3. **Repetition Count (`xN`)**: send command N times per trigger (`x1`, `x2`, `x5`).
4. **Remaining Lifespan (`rN`)**: batch auto-deletes after N triggers (`r0` = unlimited, `r5` = 5 runs).
5. **Command (`cmd`)**: slash command sent to the bot (`/claim`, `/daily`, etc.).

#### Adding Bots via One-Liner (`/addbot`):
```text
/addbot @username [1|0] [Emoji] Name. <batch_spec>
```
Examples:
- `/addbot @coolbot 1 🐥 Chicks. /claim`
- `/addbot @coolbot 1 🐥 Chicks. 02:30 /claim /crack`
- `/addbot @coolbot 1 🐥 Chicks. b1 01:00 /daily b2 03:00 x5 /tap`
- `/addbot @coolbot 1 🐥 Chicks. @14:30 /daily`
- `/addbot @coolbot 1 🐥 Chicks. 02:00 r5 /promo`

---

### ⌨️ Command Reference

- `/start`, `/menu` — Open main interactive menu.
- `/status` — View battery, queue, and active timer status.
- `/farm` — Run full collection cycle across all bots immediately.
- `/farm<ID>` — Run collection for a specific bot immediately (`/farm1`).
- `/pause` / `/resume` — Pause or resume harvest queue.
- `/rep <interval>` — Set regular status report interval (`/rep 2h`).
- `/interval<ID> <time>` — Quick change interval for bot (`/interval1 1h30m`).
- `/time<ID> <time>` — Delay bot next run or set daily schedule (`/time1 @14:00`).
- `/delbot<ID>` — Remove bot by ID (`/delbot2`).
- `/addbot` — Bot creation wizard or syntax guide.
- `/restart` — Safe restart via watchdog supervisor.
- `/exp` — Export current script file as `.txt` to Telegram.
- `/force_update` — Apply attached script file immediately.

---

### ⚡ Architectural Highlights in V2.9.2.10 ATX

- **Long Polling (20s)**: Allows CPU to enter deep sleep during idle periods, slashing battery consumption by >80%.
- **Dashboard Removal**: Completely dismantled ANSI redraw loop and background ticker in favor of clean, continuous stream logs (`log()`).
- **Scope Error Fixed**: Eliminated `local: can only be used in a function` error on line 4575.
- **Inline Keyboards Repaired**: Converted callback processing to dedicated `jq` extraction, fixing dropped buttons.
- **Built-in Bash Optimization**: Replaced repetitive `cat`, `awk`, `sed` subprocess forks with Bash parameter expansions.
- **Deep Orphan Killer**: Added `/proc` scanning and `su` invocation to terminate legacy background root processes.
- **Automatic Dependencies**: Self-installs `curl` and `jq` via `pkg`/`apt` on first startup.

---
*(C) 2026 LukesElsv — Farm Root Project*
