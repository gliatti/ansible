# pgbouncer

Installation et configuration de pgBouncer (Debian et RedHat).

## Tasks

* `install_manager.yml`: dépôt PGDG, paquet et service
* `hba_manager.yml`: fichier `pgbouncer_hba.conf` depuis `pgbouncer_hba_entries`
* `database_entries_manager.yml`: fichier `pgbouncer.ini` (toujours déployé, aussi par `install_manager.yml` avant le premier démarrage), section `[databases]` depuis `pgbouncer_database_entries`
* `userlist_manager.yml`: fichier `userlist.txt` depuis `pgbouncer_userlist`

`tasks/main.yml` enchaîne les trois gestionnaires de configuration.

## Available variables

Voir `defaults/main.yml`. Sur RedHat, `vars/RedHat.yml` remplace l'utilisateur
système (`pgbouncer`), le journal, le pid et le répertoire de socket pour
coller au RPM PGDG. La socket Unix y est placée dans `/run/pgbouncer` (et non
`/tmp`, défaut du RPM, accessible en écriture à tous) : les clients locaux
utilisent `psql -h /run/pgbouncer -p 5433`.

* `pgbouncer_packages`: paquets à installer
* `pgbouncer_sys_user` / `pgbouncer_sys_group`: propriétaire des fichiers de configuration
* `pgbouncer_inifile`, `pgbouncer_auth_file`, `pgbouncer_auth_hba_file`: chemins des fichiers
* `pgbouncer_default_port`, `pgbouncer_listen_addr`, `pgbouncer_unix_socket_directories`
* `pgbouncer_auth_type`, `pgbouncer_admin_users`
* `pgbouncer_pool_mode`, `pgbouncer_server_reset_query`, `pgbouncer_ignore_startup_parameters`
* `pgbouncer_max_client_conn`, `pgbouncer_default_pool_size`, `pgbouncer_reserve_pool_size`
* `pgbouncer_database_entries`: liste d'entrées `[databases]`
  * `forcedb`: nom exposé, `host`, `port`, `dbname`
* `pgbouncer_userlist`: liste d'utilisateurs (`name`, `password`), mot de passe en clair dans `userlist.txt` (0600) pour l'authentification SCRAM des deux côtés
* `pgbouncer_hba_entries`: même format que `postgresql_hba_entries`
