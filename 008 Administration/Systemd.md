## 🔧 Где должны лежать systemd unit-файлы?

### 📁 `/etc/systemd/system/`

Это **правильное место для системных юнитов**, созданных вручную:

* Юниты здесь доступны **всей системе** (всем пользователям).
* Службы запускаются **от имени root** или указанного пользователя (`User=...`).
* Это то место, куда systemd ожидает пользовательские системные службы (например, твой `etl.service`).

> ✅ Это **рекомендуемое место**, если ты хочешь, чтобы служба стартовала при загрузке или была частью системных служб (`systemctl start`, `enable`, `status` и т.д.)

---

### 📁 `/etc/systemd/user/`

Это — **для user units**, т.е. служб, которые запускаются **в рамках сессии конкретного пользователя**, а не на уровне системы.

* Запускаются не через обычный `systemctl`, а через `systemctl --user`.
* Работают **только при активной сессии пользователя** (если не настроено иначе).
* Не стартуют автоматически при загрузке системы, если не включён user lingering.

> ⚠️ Это **не то, что тебе нужно**, если хочешь, чтобы скрипт работал в фоновом режиме на уровне системы.

---

## 📦 Где хранятся юниты системных сервисов (PostgreSQL, ClickHouse и т.п.)?

### 📁 `/usr/lib/systemd/system/`

* Сюда ставятся **юниты, установленные через пакеты** (`rpm`, `deb` и т.д.).
* Например:

  * PostgreSQL: `/usr/lib/systemd/system/postgresql.service`
  * ClickHouse: `/usr/lib/systemd/system/clickhouse-server.service`

> 📌 **Не редактируй их прямо здесь**, если хочешь изменить — лучше копировать в `/etc/systemd/system/` и там переопределять.

---

## 💡 Итого: где твой `etl.service` должен лежать?

**✅ Рекомендуемое:**

```bash
/etc/systemd/system/etl.service
```

* Доступен всей системе.
* Запускается как системный сервис.
* Позволяет использовать `sudo systemctl start|stop|enable etl.service`.

---

Если запускать только в пользовательской сессии (например, на рабочем столе, без root), то можно использовать `~/.config/systemd/user/`, но это совсем другая история.

---

## How to

1. 🔍 Посмотреть, где лежат systemd-юниты.
2. 🛠 Переопределить системный юнит (без переписывания).
3. 📋 Отобразить все активные или установленные юниты.

---

## 🔍 1. Где лежит конкретный юнит?

Например, найти юнит `postgresql.service`:

```bash
systemctl status postgresql
```

В строке `Loaded:` будет написано, откуда загружен юнит:

```text
Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
```

---

## 🛠 2. Как переопределить системный юнит (без прямого редактирования)

Если хочешь изменить юнит PostgreSQL (или любой другой из `/usr/lib/systemd/system/`) — не редактируй его прямо.

Вместо этого:

```bash
sudo systemctl edit postgresql
```

Это создаст (или откроет) файл-оверлей:

```bash
/etc/systemd/system/postgresql.service.d/override.conf
```

Ты можешь переопределить только нужные поля. Например:

```ini
[Service]
Environment="PGDATA=/custom/postgres/data"
```

После редактирования — обязательно:

```bash
sudo systemctl daemon-reload
sudo systemctl restart postgresql
```

---

## 📋 3. Как посмотреть все юниты

### Все юниты, которые systemd знает:

```bash
systemctl list-unit-files
```

### Только активные сервисы:

```bash
systemctl list-units --type=service
```

### Найти конкретный юнит:

```bash
systemctl list-units | grep etl
systemctl list-unit-files | grep postgres
```

---

## 📍 Где systemd ищет юниты (в порядке приоритета)

```bash
/etc/systemd/system/               # пользовательские системные юниты
/run/systemd/system/              # временные (напр. сгенерированные)
/usr/lib/systemd/system/          # установленные пакетом
```

И переопределения:

```bash
/etc/systemd/system/<service>.service.d/override.conf
```

---
