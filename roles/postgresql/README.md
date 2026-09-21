# postgresql

Handle specific actions for postgresql

## Tasks

Tasks are extracted in several files, included in `tasks/main.yml`

* `cron_manager.yml`: Manage the crons files using the `cron_schedules` var and the `cron` role
* `databases_manager.yml`: Manage the postgresql databases using the `postgresql_databases` var
* `extensions_manager.yml`: Manage a list of extensions for each database using the `postgresql_databases` var
* `hba_manager.yml`: Manage the pg_hba.conf file using the `postgresql_hba_entries` var
* `repository_manager.yml`: PGDG repository (key + sources), also used by the pgbouncer and barman roles
* `install_manager.yml`: Install postgresql normalized (repository, packages, initdb, layout, shell profile)
* `layout.yml`: Backup and maintenance directories, `$PGDATA/backups` symlink
* `bash_manager.yml`: `.bashrc` and `.bash_profile` of the postgres user
* `pg_pass_manager.yml`: Manage the .pg_pass file
* `prerequisites.yml`: Manage the prerequisites for postgresql
* `privileges_manager.yml`: Manage roles privileges
* `roles_manager.yml`: Manage users

## Available variables
* `postgresql_repository_signing_key_url`: The signing key URL
* `postgresql_repository`: The repository in the postgresql source list
* `postgresql_path_bin`: Path to postgresql binaries
* `postgresql_path_lib`: Path to postgresql libraries
* `postgresql_repository_list_name`: Name of the source list for postgresql
* `postgresql_symlinks`: Symlinks to create (default: `$PGDATA/backups` -> `postgresql_pgbackup_directory`)
  * `src`: The directory of the symlink
  * `path`: Path where the symlink will be created
* `postgresql_initdb_options`: initdb options shared by Debian (through `pg_createcluster -- ...`) and RedHat
* `postgresql_pgdata_directory`: The directory of postgresql
* `postgresql_version`: The version of postgresql to install
* `postgresql_packages`: List of packages of postgresql
* `postgresql_env_vars`: The environment variables for maintenance purposes
* `postgresql_pgdata_main_directory`: The directory of the main
* `postgresql_pgbackup_directory`: The main backup directory
* `postgresql_pgbackup_directories`: The subdirectory in `postgresql_pgbackup_directory`
* `postgresql_maintenance_directories`: Maintenance directories (`dba`, `scripts`, cron logs)
* `postgresql_global_config_options`: List of all postgresql parameters to modify
  * `option`: The name of the option
  * `value`: The value of the option
* `postgresql_locale`: The locale
* `postgresql_port_number`: The port number
* `postgresql_system_database`: The name of the system database
* `postgresql_default_owner`: The default owner
* `postgresql_sys_user`: The system user used to launch postgresql service
* `postgresql_sys_group`: The group of the system user, for directory and file privileges
* `postgresql_default_extensions`: Default extension that will be installed in all databases
  * `name`: The name of the extension
  * `state`: The state of the extension
* `postgresql_databases`: List of the databases
  * `name`: The name of the database
  * `owner`: The owner of the database
  * `state`: The state of the database
  * `extensions`: List of extensions that will be installed in the database. Same as `postgresql_default_extensions` var
* `postgresql_roles`: List of roles or groups
  * `name`: The role's name
  * `password`: The role's password, in case of group don't use `password`
  * `role_attr_flags`: The role's attributes
  * `state`: The state of the role

* `postgresql_privileges`: The privileges of each role for each database
  * `database`: The database in which to grant or revoke privileges for the role. To grant or revoke privileges for a group to a role, please use the system database
  * `roles`: The role
  * `privs`: Privlieges, SELECT,UPDATE,INSERT,DELETE. If `type` is set to group don't use `privs`
  * `objs`: The objects the privileges will take place, ALL_IN_SCHEMA. Use name of the group when you want to add a role to a group
  * `schema`: The name of the schema
  * `type`: The type of objects to grant or revoke privileges, table | sequence | group
* `postgresql_hba_entries`: List of entries for each database for a pg_hba.conf file
  * `database`: The name of the database
  * `host_based_authentications`: List of all authentication for this database
    * `type`: The type of authentication
    * `user`: The groups or users
    * `address`: The client address
    * `auth_method`: The authentication method
