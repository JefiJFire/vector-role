# Ansible Role: Vector

Устанавливает [Vector](https://vector.dev) — легковесный инструмент для сбора, обработки и маршрутизации логов и метрик.  

## Requirements

- Ansible 2.10 или новее.
- Целевая система: Debian/Ubuntu.
- Доступ к [https://packages.timber.io/vector/](https://packages.timber.io/vector/) для загрузки архива (или зеркало).
- На управляемой машине должны быть установлены `tar`, `gzip`, `systemd`.

## Role Variables

В `defaults/main.yml` используется лишь одна переменная - `clickhouse`. Её необходимо переопределить на ip-адресс сервера с clickhouse.

## Dependencies

Роль не зависит от других ролей. Требуется только стандартный набор модулей Ansible: `apt`, `shell` и `template`.

## Example Playbook

```ansible
- name: Install Vector
  hosts: vector
  roles:
    - vector-role
  tags: vector
```

## License

MIT

## Author Information

Creater: Korobeynikov Vladislav

