# Роль postgres

Ставит PostgreSQL из репозиториев Ubuntu, инициализирует кластер в `postgres__data_dir`, запускает его
systemd-службой `postgres`, создаёт пользователей и базы, ставит postgres_exporter (служба `postgres-exporter`).

Служба `postgresql@<версия>-main` из пакета останавливается и отключается.

## Переменные

| Переменная | По умолчанию | Описание |
|---|---|---|
| `postgres__version` | `14` | Версия PostgreSQL (пакет `postgresql-<версия>`) |
| `postgres__data_dir` | `/data/postgres/data` | Каталог данных |
| `postgres__opts` | `listen_addresses`, `port` | Словарь параметров `postgresql.conf`. Значение пишется в файл как есть, строки со спецсимволами нужно брать в одинарные кавычки: `"'*'"` |
| `postgres__hba` | peer для local, scram-sha-256 для 127.0.0.1 | Список правил `pg_hba.conf`: `type`, `database`, `user`, `address`, `method` |
| `postgres__users` | `[]` | Список пользователей: `name`, `password`, `role_attr_flags` |
| `postgres__databases` | `[]` | Список баз: `name`, `owner`, `encoding` |
| `postgres__exporter_version` | `0.17.1` | Версия postgres_exporter |
| `postgres__exporter_port` | `9187` | Порт экспортера |
| `postgres__exporter_password` | `""` | Пароль роли `postgres_exporter` (член `pg_monitor`) |

Пароли задавайте через ansible-vault.

## Зависимости

Коллекция `community.postgresql`.

## Пример плейбука

```yaml
- name: Install PostgreSQL on vm1
  hosts: vm1
  vars:
    postgres__opts:
      listen_addresses: "'*'"
      port: 5432
      shared_buffers: 256MB
    postgres__users:
      - name: app
        password: "{{ vault_postgres_app_password }}"
    postgres__databases:
      - name: appdb
        owner: app
    postgres__exporter_password: "{{ vault_postgres_exporter_password }}"
  roles:
    - postgres
```
