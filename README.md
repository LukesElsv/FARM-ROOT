# 🌾 FARM ROOT — README

[⬇️ English version below](#english-version)

---

## 🇷🇺 Русская версия

### Что такое FARM ROOT

**FARM ROOT** — автономная Telegram-ферма для Android, работающая в Termux. Она запускается в фоне телефона, по расписанию открывает нужных ботов в Telegram, отправляет заранее заданные команды и собирает награды — карточки, ежедневные бонусы, тапы, часовые дропы и всё, что нужно «дёргать» раз в какое-то время.

Управление — из Telegram, кнопками. Никакой терминал после установки не нужен: главное меню, настройка ботов, таймеры, отчёты, логи, обновление — всё через инлайн-панель.

### Зачем это нужно

Если у вас есть боты, в которых нужно раз в 30 минут, час или три часа нажимать одну и ту же кнопку/команду и получать награду — эта рутина полностью автоматизируется.

Ферма сама:

- знает, каких ботов и когда дёргать;
- открывает Telegram, вставляет команду, отправляет;
- при неудаче — 3 попытки, при совсем плохом раскладе — перезапуск Telegram и заново;
- ведёт цветовой отчёт: 🟢 с первого раза, 🟡 со 2–3, 🔴 провал, ⚪ нет данных;
- следит за батареей: если заряд критически упал, предупредит;
- перезапускается после сбоев;
- присылает вам короткие отчёты в личку.

Никаких сторонних серверов и «облаков». Всё локально.

---

### ⚠️ Требования

Читайте внимательно — от окружения напрямую зависит стабильность.

#### 1. Root — **обязателен**

Скрипт изначально писался **под root** и без него не тестировался. Root нужен для:

- корректного чтения батареи (`dumpsys battery`);
- надёжного `wake-lock` и OOM-защиты;
- точного выполнения макроса (открытие Telegram, ввод текста, отправка);
- работы с системными сервисами (`uiautomator`, `input`, `settings`).

**Протестировано на:** **Magisk** актуальной версии.

Возможно, будет работать и на других root-решениях — **KernelSU**, **APatch**, **SuperSU** — но такое тестирование не проводилось.

❌ **Без root — не работает.**

#### 2. Стоковый Telegram — **обязательно**

Разработка и тестирование шли **только под официальный клиент**.

- ✅ Подходит: обычный Telegram из Google Play / с официального сайта.
- ❌ Не подходит: Telegram X, Nekogram, Plus Messenger, Nicegram, Turrit и любые модификации.

#### 3. Прошивка AOSP / Pixel-like — **рекомендуется**

Стабильно работает на:

- LineageOS
- RisingOS
- crDroid
- и другие чистые AOSP-сборки с Pixel/Nexus-лаунчером

❌ Не тестировалось: MIUI / HyperOS, One UI, EMUI, ColorOS, Funtouch — они агрессивно убивают фоновые процессы и ломают wake-lock.

#### 4. Termux — **только из F-Droid**

Версия из Google Play устарела. Скачивать строго тут: https://f-droid.org/packages/com.termux/

#### 5. Android

Android 8.0 и выше. iOS **не поддерживается**.

---

### Установка

**Шаг 0. Root.** Установите Magisk по официальной инструкции и проверьте в Termux:

```bash
su -c id
```

Должна вернуться строка с `uid=0(root)`.

**Шаг 1. Termux.** Установите Termux из F-Droid. Откройте и выполните:

```bash
pkg update -y && pkg install -y bash curl jq coreutils
```

**Шаг 2. Бот.** В Telegram откройте `@BotFather`, отправьте `/newbot`, следуйте инструкциям. Сохраните **токен** вида `1234567890:ABCdef...`.

**Шаг 3. Chat ID.** Откройте `@userinfobot`, нажмите Start. Он пришлёт `Id: 123456789` — это ваш Chat ID.

**Шаг 4. Скрипт.** Положите файл релиза в домашнюю папку Termux.

**Шаг 5. Запуск.**

```bash
bash ~/farm_V1.8.5_20261001.txt
```

Первый запуск:

- создаст структуру `~/storage/shared/farmvers/database/`;
- спросит токен и Chat ID (либо впишите их в начало скрипта в `TG_TOKEN=""` и `TG_CHAT_ID=""`);
- сохранит их в `state/tg_config.txt` (права `600`);
- пришлёт вам главное меню в Telegram.

**Шаг 6.** Работайте из Telegram. Терминал можно свернуть.

---

### Функционал

- Управление несколькими Telegram-ботами с разными интервалами
- Автосбор по расписанию — полный цикл или отдельный бот
- Ручной выборочный запуск одного бота одной кнопкой
- До 3 попыток отправки, включая перезапуск Telegram при сбое
- Цветовая система оценки результата (🟢 / 🟡 / 🔴 / ⚪)
- Очередь ботов: перестановка, вкл/выкл
- Гибкие таймеры: отложить, сбросить, изменить интервал
- Настраиваемая периодичность отчётов
- Живой лог макроса и история последних сборов
- Логи крашей с причиной на RU/EN
- Доктор: авто-диагностика и авто-восстановление
- Обновление из Telegram + автооткат при сбоях
- Wake-lock и OOM-защита
- Экономия заряда, предупреждение о низкой батарее
- Полностью двуязычный интерфейс RU / EN

---

### Стоковые боты в очереди

При первом запуске автоматически создаются три стоковых бота. Любого можно переименовать, отключить, удалить или заменить.

| ID | Имя | Username | Команда | Интервал |
|----|-----|----------|---------|----------|
| 1  | 📱 Телефоны    | phonegetcardsbot  | /tcard | 9030 сек  |
| 2  | 💻 Компьютеры  | createpcpartsbot  | /part  | 10830 сек |
| 3  | 🐰 Кролики     | cabbitget_bot     | Крол   | 3630 сек  |

---

### Команды — полный справочник с примерами

#### 🎮 Управление циклом

**`/farm`** — запустить полный цикл по всем включённым ботам.

Примеры:
```
/farm
```

**`/farm<ID>`** — выборочный запуск одного бота по ID.

Примеры:
```
/farm1
/farm2
/farm5
```

**`/pause`** (синоним `/stop`) — поставить автосбор на паузу. Все остальные функции работают.

Примеры:
```
/pause
/stop
```

**`/start_farm`** (синоним `/resume`) — снять паузу, возобновить автосбор.

Примеры:
```
/start_farm
/resume
```

**`/res`** (синоним `/reset`) — сбросить таймеры всех ботов к базовому интервалу.

Примеры:
```
/res
/reset
```

**`/res<ID>`** — сбросить таймер конкретного бота.

Примеры:
```
/res1
/res3
```

---

#### 📋 Очередь и боты

**`/queue`** — открыть меню очереди: включить/выключить бота, переместить вверх/вниз, открыть настройки ⚙️.

Примеры:
```
/queue
```

**`/addbot`** — добавить нового бота.

Синтаксис:
```
/addbot @username 02:30 1 🐥 Цыплята. /command
```

Где:
1. `@username` — юзернейм бота (`@` необязателен)
2. `02:30` — интервал в формате `ЧЧ:ММ` (можно `90m`, `5400`)
3. `1` или `0` — можно ли менять интервал через команду
4. `🐥 Цыплята` — эмодзи и имя
5. `..` — **точка как разделитель** имени и команды
6. `/command` — команда, которую шлём боту

Если точки нет — команда по умолчанию `/start`.

Примеры:
```
/addbot mybot 01:30 1 🐥 Цыплята. /claim
/addbot @mybot 00:30 0 🐰 Кролики
/addbot coolbot 2h 1 💎 Алмазы. /daily
/addbot farmbot 90m 1 ⭐ Звёзды. /bonus
/addbot hourlybot 3600 0 🎁 Подарки
```

**`/botname<ID>`** — переименовать бота (эмодзи + имя).

Примеры:
```
/botname1 🐥 Цыплята
/botname3 💎 Алмазы
```

**`/botcmd<ID>`** — изменить команду отправки.

Примеры:
```
/botcmd1 /daily
/botcmd2 /claim
/botcmd3 Крол
```

**`/botuser<ID>`** — сменить username бота.

Примеры:
```
/botuser1 @newbot
/botuser2 @anotherbot
```

**`/boticon<ID>`** — сменить только эмодзи, имя не трогать.

Примеры:
```
/boticon1 🐥
/boticon3 💎
```

**`/delbot<ID>`** — удалить бота. Стоковые удаляются с подтверждением.

Примеры:
```
/delbot1
/delbot5
```

---

#### ⏱ Таймеры

**`/timers`** — открыть меню всех таймеров.

Примеры:
```
/timers
```

**`/time<ID> ЧЧ:ММ`** — отложить сбор конкретного бота на указанное время.

Примеры:
```
/time1 00:15
/time2 01:00
/time3 90m
```

**`/time ЧЧ:ММ`** — отложить сбор выбранного бота.

Примеры:
```
/time 00:30
/time 2h
```

**`/interval<ID> ЧЧ:ММ`** — изменить базовый интервал конкретного бота.

Примеры:
```
/interval1 01:30
/interval2 02:00
/interval3 30m
```

**`/interval ЧЧ:ММ`** — изменить интервал выбранного бота.

Примеры:
```
/interval 01:00
/interval 90m
```

---

#### 🔔 Отчёты

**`/rep ЧЧ:ММ`** — периодичность авто-отчётов.

Поддерживаются три формата: `ЧЧ:ММ`, минуты (`30m`), секунды (`3600`).

Примеры:
```
/rep 01:00
/rep 30m
/rep 3600
/rep 02:00
```

---

#### 📊 Логи и статус

**`/status`** — текущий статус фермы: заряд батареи, аптайм, интервал отчётов, состояние (пауза/активна), очередь ботов и таймеры.

Примеры:
```
/status
```

**`/macrolog`** (синоним `/mlog`) — лог макроса: список ботов, результаты последнего сбора, история последних 10 операций.

Примеры:
```
/macrolog
/mlog
```

**`/logs`** (синоним `/macro`) — логи крашей с пагинацией.

Примеры:
```
/logs
/macro
```

---

#### 📦 Обновление

**`/update`** — открыть меню обновления.

Примеры:
```
/update
```

**`/versions`** — список сохранённых версий и возможность откатиться.

Примеры:
```
/versions
```

**`/saveversion`** — сохранить текущую версию как бэкап.

Примеры:
```
/saveversion
```

**`/exp`** — отправить текущий скрипт вам в Telegram (экспорт).

Примеры:
```
/exp
```

---

#### 🩺 Диагностика и язык

**`/diag`** — самодиагностика: зависимости, синтаксис, доступ к данным, Telegram API, root, батарея, устаревшие PID.

Примеры:
```
/diag
```

**`/language`** (синоним `/lang`) — сменить язык интерфейса (RU / EN).

Примеры:
```
/language
/lang
```

**`/start`** (синоним `/menu`) — обновить главное меню.

Примеры:
```
/start
/menu
```

---

### Служебные файлы

Все данные лежат в `~/storage/shared/farmvers/database/`:

- `bots/` — настройки каждого бота
- `queue/` — очередь
- `timers/` — таймеры
- `logs/` — логи (verbose, macro, crashes, doctor)
- `settings/` — интервал отчётов, next_report
- `state/` — PID, конфиг Telegram, состояние батареи, offset
- `pending/` — отложенные действия
- `collections/` — история сборов
- `versions/` — сохранённые версии скрипта

Креды (`state/tg_config.txt`) хранятся отдельно от скрипта и **не попадают** в публикуемый код.

---

### Обновление

Отправьте боту `.txt` файл новой версии — бот сам проверит синтаксис, прогонит selftest, сохранит бэкап, установит и перезапустится. Креды переносятся автоматически.

Или через **📦 Обновление → 📥 Загрузить** и прикрепить файл.

---

### Предупреждение

Программа для **личного использования**. Не предназначена для нарушения правил Telegram, обхода ограничений сторонних ботов, спама и незаконных действий. Используйте на свой страх и риск.

Автор не несёт ответственности за блокировки аккаунтов и любые последствия.

---

### Обратная связь

- Telegram (Основной): `@LKS2009`
- Telegram (Резервный): `@stg_lucasew`
- Email: `diamondproto.help@gmail.com`
- TikTok: `@lukaelsv`

**Автор:** LKS

---

<a id="english-version"></a>

## 🇬🇧 English Version

### What FARM ROOT is

**FARM ROOT** is an autonomous Telegram farm for Android, running inside Termux. It launches in the background of your phone, opens the right bots in Telegram on schedule, sends pre-configured commands, and collects rewards — cards, daily bonuses, taps, hourly drops, and anything else that needs to be "poked" every so often.

Control happens from Telegram, with buttons. No terminal needed after install: main menu, bot setup, timers, reports, logs, updates — all through an inline panel.

### Why you need it

If you have bots where you need to tap the same button or send the same command every 30 minutes, hour, or three hours to claim a reward — that routine can be fully automated.

The farm:

- knows which bots to ping and when;
- opens Telegram, pastes the command, sends it;
- retries up to 3 times, and on a bad day — restarts Telegram and tries again;
- keeps a color-coded report: 🟢 first try, 🟡 second/third, 🔴 fail, ⚪ no data;
- monitors battery: warns you if it drops critically;
- restarts after crashes;
- sends you short reports in your DM.

No third-party servers, no "clouds". Everything is local.

---

### ⚠️ Requirements

Read carefully — stability directly depends on the environment.

#### 1. Root — **mandatory**

The script was written **for root** and has never been tested without it. Root is used for:

- correct battery reading (`dumpsys battery`);
- reliable `wake-lock` and OOM protection;
- accurate macro execution (opening Telegram, pasting text, sending);
- access to system services (`uiautomator`, `input`, `settings`).

**Tested on:** **Magisk (latest version)**.

May work with other root solutions — **KernelSU**, **APatch**, **SuperSU** — but such testing has not been performed.

❌ **Without root — it won't work.**

#### 2. Stock Telegram — **mandatory**

Development and testing were done **only against the official client**.

- ✅ Supported: regular Telegram from Google Play / official site.
- ❌ Not supported: Telegram X, Nekogram, Plus Messenger, Nicegram, Turrit and any modifications.

#### 3. AOSP / Pixel-like firmware — **recommended**

Runs stably on:

- LineageOS
- RisingOS
- crDroid
- and other clean AOSP builds with Pixel/Nexus launcher

❌ Not tested: MIUI / HyperOS, One UI, EMUI, ColorOS, Funtouch — they aggressively kill background processes and break wake-locks.

#### 4. Termux — **F-Droid only**

The Google Play version is outdated. Download strictly from: https://f-droid.org/packages/com.termux/

#### 5. Android

Android 8.0+. iOS is **not supported**.

---

### Installation

**Step 0. Root.** Install Magisk following the official guide and verify in Termux:

```bash
su -c id
```

You should get a line containing `uid=0(root)`.

**Step 1. Termux.** Install Termux from F-Droid. Open it and run:

```bash
pkg update -y && pkg install -y bash curl jq coreutils
```

**Step 2. Bot.** In Telegram open `@BotFather`, send `/newbot`, follow prompts. Save the **token** like `1234567890:ABCdef...`.

**Step 3. Chat ID.** Open `@userinfobot`, press Start. It will reply `Id: 123456789` — this is your Chat ID.

**Step 4. Script.** Put the release file into your Termux home folder.

**Step 5. Run.**

```bash
bash ~/farm_V1.8.5_20261001.txt
```

First run:

- creates the `~/storage/shared/farmvers/database/` structure;
- asks for token and Chat ID (or you can paste them at the top of the script in `TG_TOKEN=""` and `TG_CHAT_ID=""`);
- saves them to `state/tg_config.txt` (permissions `600`);
- sends the main menu to your Telegram.

**Step 6.** Work from Telegram. Terminal can be minimized.

---

### Features

- Manage multiple Telegram bots with different intervals
- Scheduled autocollect — full cycle or a single bot
- Manual selective run of a bot with one button
- Up to 3 attempts, including Telegram restart on failure
- Color-coded result scoring (🟢 / 🟡 / 🔴 / ⚪)
- Queue with reordering and enable/disable
- Flexible timers: delay, reset, change interval
- Configurable report frequency
- Live macro log and history of recent runs
- Crash logs with reason in RU/EN
- Doctor: auto-diagnostics and auto-recovery
- Telegram-based update + auto-rollback on failures
- Wake-lock and OOM protection
- Power saving, low battery warnings
- Fully bilingual interface RU / EN

---

### Default stock bots

On first launch, three stock bots are created. Each can be renamed, disabled, deleted or replaced.

| ID | Name           | Username          | Command | Interval |
|----|----------------|-------------------|---------|----------|
| 1  | 📱 Phones      | phonegetcardsbot  | /tcard  | 9030 s   |
| 2  | 💻 Computers   | createpcpartsbot  | /part   | 10830 s  |
| 3  | 🐰 Rabbits     | cabbitget_bot     | Крол    | 3630 s   |

---

### Commands — full reference with examples

#### 🎮 Cycle control

**`/farm`** — run full cycle over all enabled bots.

Examples:
```
/farm
```

**`/farm<ID>`** — selective run of a single bot by ID.

Examples:
```
/farm1
/farm2
/farm5
```

**`/pause`** (alias `/stop`) — pause autocollect. All other features keep working.

Examples:
```
/pause
/stop
```

**`/start_farm`** (alias `/resume`) — resume autocollect.

Examples:
```
/start_farm
/resume
```

**`/res`** (alias `/reset`) — reset all bot timers to their base interval.

Examples:
```
/res
/reset
```

**`/res<ID>`** — reset a specific bot timer.

Examples:
```
/res1
/res3
```

---

#### 📋 Queue and bots

**`/queue`** — open the queue menu: enable/disable, move up/down, open ⚙️ settings.

Examples:
```
/queue
```

**`/addbot`** — add a new bot.

Syntax:
```
/addbot @username 02:30 1 🐥 Chicks. /command
```

Where:
1. `@username` — bot username (`@` optional)
2. `02:30` — interval in `HH:MM` (`90m`, `5400` also work)
3. `1` or `0` — whether the interval can be changed via command
4. `🐥 Chicks` — emoji and name
5. `..` — **dot as separator** between name and command
6. `/command` — command to send to the bot

If there's no dot — the default is `/start`.

Examples:
```
/addbot mybot 01:30 1 🐥 Chicks. /claim
/addbot @mybot 00:30 0 🐰 Rabbits
/addbot coolbot 2h 1 💎 Diamonds. /daily
/addbot farmbot 90m 1 ⭐ Stars. /bonus
/addbot hourlybot 3600 0 🎁 Gifts
```

**`/botname<ID>`** — rename bot (emoji + name).

Examples:
```
/botname1 🐥 Chicks
/botname3 💎 Diamonds
```

**`/botcmd<ID>`** — change the sending command.

Examples:
```
/botcmd1 /daily
/botcmd2 /claim
/botcmd3 Крол
```

**`/botuser<ID>`** — change bot username.

Examples:
```
/botuser1 @newbot
/botuser2 @anotherbot
```

**`/boticon<ID>`** — change only the emoji, keep the name.

Examples:
```
/boticon1 🐥
/boticon3 💎
```

**`/delbot<ID>`** — delete a bot. Stock bots ask for confirmation.

Examples:
```
/delbot1
/delbot5
```

---

#### ⏱ Timers

**`/timers`** — open the timers menu.

Examples:
```
/timers
```

**`/time<ID> HH:MM`** — delay a specific bot.

Examples:
```
/time1 00:15
/time2 01:00
/time3 90m
```

**`/time HH:MM`** — delay the selected bot.

Examples:
```
/time 00:30
/time 2h
```

**`/interval<ID> HH:MM`** — change base interval of a specific bot.

Examples:
```
/interval1 01:30
/interval2 02:00
/interval3 30m
```

**`/interval HH:MM`** — change interval of the selected bot.

Examples:
```
/interval 01:00
/interval 90m
```

---

#### 🔔 Reports

**`/rep HH:MM`** — set auto-report interval.

Three formats supported: `HH:MM`, minutes (`30m`), seconds (`3600`).

Examples:
```
/rep 01:00
/rep 30m
/rep 3600
/rep 02:00
```

---

#### 📊 Logs and status

**`/status`** — current farm status: battery, uptime, report interval, state (paused/active), bot queue and timers.

Examples:
```
/status
```

**`/macrolog`** (alias `/mlog`) — macro log: list of bots, last run results, last 10 operations.

Examples:
```
/macrolog
/mlog
```

**`/logs`** (alias `/macro`) — crash logs with pagination.

Examples:
```
/logs
/macro
```

---

#### 📦 Update

**`/update`** — open the update menu.

Examples:
```
/update
```

**`/versions`** — list of saved versions with rollback.

Examples:
```
/versions
```

**`/saveversion`** — save the current version as a backup.

Examples:
```
/saveversion
```

**`/exp`** — export the current script to your Telegram.

Examples:
```
/exp
```

---

#### 🩺 Diagnostics and language

**`/diag`** — self-diagnostics: dependencies, syntax, data access, Telegram API, root, battery, stale PIDs.

Examples:
```
/diag
```

**`/language`** (alias `/lang`) — change interface language (RU / EN).

Examples:
```
/language
/lang
```

**`/start`** (alias `/menu`) — refresh the main menu.

Examples:
```
/start
/menu
```

---

### Service files

All data lives in `~/storage/shared/farmvers/database/`:

- `bots/` — per-bot settings
- `queue/` — queue order
- `timers/` — timers
- `logs/` — verbose, macro, crashes, doctor
- `settings/` — report interval, next_report
- `state/` — PIDs, Telegram config, battery state, offset
- `pending/` — pending actions
- `collections/` — collection history
- `versions/` — saved script versions

Credentials (`state/tg_config.txt`) are stored separately from the script and **never** land in the published code.

---

### Updating

Send the bot the new `.txt` file — it will verify syntax, run selftest, back up, install, and restart. Credentials are migrated automatically.

Or via **📦 Update → 📥 Upload** and attach the file.

---

### Disclaimer

For **personal use only**. Not intended for violating Telegram rules, bypassing limitations of third-party bots, spam, or illegal actions. Use at your own risk.

The author is not responsible for account blocks or any consequences.

---

### Feedback

- Telegram (Main): `@LKS2009`
- Backup Telegram: `@stg_lucasew`
- Email: `diamondproto.help@gmail.com`
- TikTok: `@lukaelsv`

**Author:** LKS
