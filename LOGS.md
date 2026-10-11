### Установка коллекций

```console
$ ansible-galaxy collection install -r requirements.yml
Starting galaxy collection install process
Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using `--force`.
```

### Первый прогон

```console
$ ansible-playbook postgres.play.yml

PLAY [Install PostgreSQL on vm1] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [vm1]

TASK [postgres : Install Postgres packages] ************************************
ok: [vm1]

TASK [postgres : Stop default cluster] *****************************************
ok: [vm1]

TASK [postgres : Create /data] *************************************************
changed: [vm1]

TASK [postgres : Create data directories] **************************************
changed: [vm1] => (item=/data/postgres)
changed: [vm1] => (item=/data/postgres/data)

TASK [postgres : Init database cluster] ****************************************
changed: [vm1]

TASK [postgres : Edit postgresql.conf] *****************************************
changed: [vm1] => (item=listen_addresses)
changed: [vm1] => (item=port)
changed: [vm1] => (item=max_connections)
changed: [vm1] => (item=shared_buffers)
changed: [vm1] => (item=password_encryption)

TASK [postgres : Copy pg_hba.conf] *********************************************
changed: [vm1]

TASK [postgres : Copy postgres systemd service] ********************************
changed: [vm1]

TASK [postgres : Flush handlers] ***********************************************

RUNNING HANDLER [postgres : Daemon reload] *************************************
ok: [vm1]

RUNNING HANDLER [postgres : Restart Postgres] **********************************
changed: [vm1]

RUNNING HANDLER [postgres : Reload Postgres] ***********************************
changed: [vm1]

TASK [postgres : Start postgres systemd service] *******************************
changed: [vm1]

TASK [postgres : Check Postgres] ***********************************************
ok: [vm1]

TASK [postgres : Create users] *************************************************
changed: [vm1] => (item=None)
changed: [vm1] => (item=None)
changed: [vm1]

TASK [postgres : Create databases] *********************************************
changed: [vm1] => (item=appdb)
changed: [vm1] => (item=reports)

TASK [postgres : Create exporter role] *****************************************
changed: [vm1]

TASK [postgres : Grant pg_monitor to exporter role] ****************************
changed: [vm1]

TASK [postgres : Create download dir] ******************************************
ok: [vm1 -> localhost]

TASK [postgres : Download exporter] ********************************************
ok: [vm1 -> localhost]

TASK [postgres : Find exporter executable] *************************************
ok: [vm1 -> localhost]

TASK [postgres : Register path to executable] **********************************
ok: [vm1 -> localhost]

TASK [postgres : Propagate fact] ***********************************************
ok: [vm1]

TASK [postgres : Copy exporter executable] *************************************
ok: [vm1]

TASK [postgres : Copy exporter connection settings] ****************************
ok: [vm1]

TASK [postgres : Copy postgres-exporter systemd service] ***********************
changed: [vm1]

TASK [postgres : Start postgres-exporter systemd service] **********************
changed: [vm1]

RUNNING HANDLER [postgres : Daemon reload] *************************************
ok: [vm1]

RUNNING HANDLER [postgres : Restart Postgres exporter] *************************
changed: [vm1]

PLAY RECAP *********************************************************************
vm1                        : ok=29   changed=16   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

### Повторный прогон

```console
$ ansible-playbook postgres.play.yml

PLAY [Install PostgreSQL on vm1] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [vm1]

TASK [postgres : Install Postgres packages] ************************************
ok: [vm1]

TASK [postgres : Stop default cluster] *****************************************
ok: [vm1]

TASK [postgres : Create /data] *************************************************
ok: [vm1]

TASK [postgres : Create data directories] **************************************
ok: [vm1] => (item=/data/postgres)
ok: [vm1] => (item=/data/postgres/data)

TASK [postgres : Init database cluster] ****************************************
ok: [vm1]

TASK [postgres : Edit postgresql.conf] *****************************************
ok: [vm1] => (item=listen_addresses)
ok: [vm1] => (item=port)
ok: [vm1] => (item=max_connections)
ok: [vm1] => (item=shared_buffers)
ok: [vm1] => (item=password_encryption)

TASK [postgres : Copy pg_hba.conf] *********************************************
ok: [vm1]

TASK [postgres : Copy postgres systemd service] ********************************
ok: [vm1]

TASK [postgres : Flush handlers] ***********************************************

TASK [postgres : Start postgres systemd service] *******************************
ok: [vm1]

TASK [postgres : Check Postgres] ***********************************************
ok: [vm1]

TASK [postgres : Create users] *************************************************
ok: [vm1] => (item=None)
ok: [vm1] => (item=None)
ok: [vm1]

TASK [postgres : Create databases] *********************************************
ok: [vm1] => (item=appdb)
ok: [vm1] => (item=reports)

TASK [postgres : Create exporter role] *****************************************
ok: [vm1]

TASK [postgres : Grant pg_monitor to exporter role] ****************************
ok: [vm1]

TASK [postgres : Create download dir] ******************************************
ok: [vm1 -> localhost]

TASK [postgres : Download exporter] ********************************************
ok: [vm1 -> localhost]

TASK [postgres : Find exporter executable] *************************************
ok: [vm1 -> localhost]

TASK [postgres : Register path to executable] **********************************
ok: [vm1 -> localhost]

TASK [postgres : Propagate fact] ***********************************************
ok: [vm1]

TASK [postgres : Copy exporter executable] *************************************
ok: [vm1]

TASK [postgres : Copy exporter connection settings] ****************************
ok: [vm1]

TASK [postgres : Copy postgres-exporter systemd service] ***********************
ok: [vm1]

TASK [postgres : Start postgres-exporter systemd service] **********************
ok: [vm1]

PLAY RECAP *********************************************************************
vm1                        : ok=24   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

## Проверки

### Службы активны

```console
$ ansible vm1 -b -m command -a 'systemctl is-active postgres postgres-exporter'
vm1 | CHANGED | rc=0 >>
active
active
```

### Автозапуск включён

```console
$ ansible vm1 -b -m command -a 'systemctl is-enabled postgres postgres-exporter'
vm1 | CHANGED | rc=0 >>
enabled
enabled
```

### SELECT 1

```console
$ ansible vm1 -b --become-user postgres -m command -a 'psql -p 5434 -c "SELECT 1"'
vm1 | CHANGED | rc=0 >>
 ?column? 
----------
        1
(1 row)
```

### Базы данных (\l)

```console
$ ansible vm1 -b --become-user postgres -m command -a 'psql -p 5434 -c "\l"'
vm1 | CHANGED | rc=0 >>
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 appdb     | app      | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 reports   | analyst  | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(5 rows)
```

### Пользователи (\du)

```console
$ ansible vm1 -b --become-user postgres -m command -a 'psql -p 5434 -c "\du"'
vm1 | CHANGED | rc=0 >>
                                         List of roles
     Role name     |                         Attributes                         |  Member of   
-------------------+------------------------------------------------------------+--------------
 analyst           | Create DB                                                  | {}
 app               |                                                            | {}
 postgres          | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 postgres_exporter |                                                            | {pg_monitor}
```

### Метрики exporter (pg_up)

```console
$ ansible vm1 -m uri -a 'url=http://127.0.0.1:9187/metrics return_content=true' | grep 'pg_up 1'
pg_up 1
```
