# mariadb

[MariaDB](https://mariadb.org) template for Runway. Relational database,
community fork of MySQL, compatible with MySQL clients.

Single instance with persistence on a volume. No replication, no failover.

## Deploy

WIP

## Inputs

| Input | Default | Notes |
| --- | --- | --- |
| `image_tag` | `12.3` | LTS lines: `12.3` (current), `11.8`, `11.4`, `10.11` |
| `database_name` | `app` | Created on first start |
| `username` | `app` | Full access to `database_name` only |

## Connecting

Not routed — reachable only from inside your organization's network. The
credentials are in the app config:

```sh
runway app config ls --app my-db
```

Connect as the application user:

```
mysql://<username>:<MARIADB_PASSWORD>@my-db.my-db.svc.cluster.local:3306/<database_name>
```

Use `root` with `MARIADB_ROOT_PASSWORD` for admin tasks only.

## Notes

- **Passwords only apply on the first start**, when the data directory is
  empty. Changing them in the config later does nothing in MariaDB — it just
  leaves your clients with a value that no longer works. Rotate over a
  connection instead:

  ```sql
  ALTER USER 'app'@'%' IDENTIFIED BY '<new>';
  ```

  Then update the config to match.

- **The template sets no `command`.** The image entrypoint reads the
  `MARIADB_*` variables and creates the database and users before starting
  `mariadbd`. Overriding the command skips that.

- **Upgrades run on start** (`MARIADB_AUTO_UPGRADE=1`). Take a backup before
  raising `image_tag`. Downgrades are not supported.

- No lockdown: the server writes its socket to `/run/mysqld`.

## License

MariaDB is GPL-2.0. This template is maintained by Runway.
