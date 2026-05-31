# Ansible Role: vector

Устанавливает и настраивает [Vector](https://vector.dev) на Debian/Ubuntu и RedHat/Rocky Linux хостах.

## Требования

- Ansible Core >= 2.15
- Python >= 3.10
- Docker >= 24.0 (для molecule-тестов)
- Поддерживаемые ОС:
  - Debian 11 (Bullseye)
  - Ubuntu 20.04 (Focal), 22.04 (Jammy)
  - Rocky Linux 8

## Переменные роли

| Переменная | Значение по умолчанию | Описание |
|---|---|---|
| `vector_version` | `0.55.0` | Версия Vector (GitHub releases) |
| `clickhouse` | `localhost` | Хост ClickHouse для sink |
| `vector_service_state` | `started` | Состояние сервиса |
| `vector_service_enabled` | `true` | Автозапуск сервиса |
| `vector_user` | `vector` | Системный пользователь |
| `vector_group` | `vector` | Системная группа |
| `vector_config_dir` | `/etc/vector` | Директория конфигов |
| `vector_data_dir` | `/var/lib/vector` | Директория данных |
| `vector_log_dir` | `/var/log/vector` | Директория логов |
| `vector_config_file` | `/etc/vector/vector.toml` | Путь к конфиг-файлу |

## Конфигурация Vector

Роль деплоит конфиг из шаблона `templates/vector.toml.j2`:

```toml
[sources.journald]
type = "journald"
include_units = []

[sinks.clickhouse]
type = "clickhouse"
inputs = ["journald"]
endpoint = "http://<clickhouse>:8123"
database = "logs"
table = "syslog"
compression = "gzip"
```

Для изменения хоста ClickHouse переопределите переменную:

```yaml
# group_vars/all.yml или inventory
clickhouse: "my-clickhouse-host"
```

## Установка

**Debian/Ubuntu** — скачивается `.deb` пакет с GitHub releases и устанавливается через `apt`.

**Rocky Linux / RedHat** — скачивается статически слинкованный musl-бинарник с GitHub releases (не зависит от версии системной glibc), systemd unit создаётся вручную.

## Использование

```yaml
# site.yml
- hosts: all
  become: true
  roles:
    - role: vector
```

```yaml
# group_vars/all.yml
clickhouse: "clickhouse.internal"
```

## Тестирование через Molecule

### Установка зависимостей

```bash
python3 -m venv ~/.venv/molecule
source ~/.venv/molecule/bin/activate
pip install -r requirements.txt
ansible-galaxy collection install -r requirements.yml
```

### Запуск тестов

```bash
cd roles/vector

# Полный цикл
molecule test

# Пошагово
molecule create       # поднять контейнеры
molecule prepare      # подготовить окружение
molecule converge     # применить роль
molecule verify       # запустить тесты
molecule idempotency  # проверить идемпотентность
molecule destroy      # удалить контейнеры

# Зайти внутрь контейнера для отладки
molecule login --host ubuntu-focal
```

### Тестовые платформы

| Контейнер | Образ |
|---|---|
| ubuntu-focal | geerlingguy/docker-ubuntu2004-ansible |
| ubuntu-latest | geerlingguy/docker-ubuntu2204-ansible |
| debian-bullseye | geerlingguy/docker-debian11-ansible |
| rockylinux-8 | geerlingguy/docker-rockylinux8-ansible |

### Что проверяют тесты (verify.yml)

- ✅ Бинарник `/usr/bin/vector` присутствует и исполняемый
- ✅ `vector --version` отрабатывает корректно
- ✅ Конфиг `/etc/vector/vector.toml` создан с правильным владельцем и правами `0640`
- ✅ Конфиг содержит секции `sources`, `sinks`, `journald`, `clickhouse`
- ✅ Директории `/etc/vector` и `/var/lib/vector` существуют
- ✅ Сервис `vector` запущен и включён в автозапуск
- ✅ Процесс `vector` присутствует в списке процессов
- ✅ Идемпотентность — повторный запуск не даёт `changed`

## Структура роли

```
roles/vector/
├── defaults/
│   └── main.yml          # переменные по умолчанию
├── handlers/
│   └── main.yml          # restart / reload vector / reload systemd
├── meta/
│   └── main.yml          # метаданные Galaxy
├── molecule/
│   └── default/
│       ├── molecule.yml  # конфиг Molecule (docker, 4 платформы)
│       ├── prepare.yml   # подготовка контейнеров
│       ├── converge.yml  # применение роли
│       └── verify.yml    # 7 групп assert-ов
├── tasks/
│   ├── main.yml          # точка входа
│   ├── install.yml       # установка (deb/musl)
│   ├── configure.yml     # деплой конфига из шаблона
│   └── service.yml       # управление сервисом
└── templates/
    └── vector.toml.j2    # конфиг Vector
```

## Версионирование

| Версия | Описание |
|---|---|
| `1.1.0` | Мульти-дистрибутивная поддержка, расширенные molecule-тесты |
| `1.0.0` | Начальный релиз |

## Семантическое версионирование и git-теги

```bash
git add .
git commit -m "feat: add multi-distro support and extended molecule tests"
git tag -a v1.1.0 -m "Release v1.1.0"
git push && git push --tags
```
