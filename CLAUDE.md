# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The Ansible collection **`gliatti.devops`** (French-language task names and comments) for provisioning PostgreSQL database servers and their ecosystem: Barman (backup), pgBouncer (pooling), ldap2pg (LDAP-driven role sync), plus supporting system roles. Collection metadata lives in `galaxy.yml` (namespace `gliatti`, name `devops`, deps `community.general`/`community.postgresql`, `build_ignore` for dev artifacts); playbooks live in `playbooks/`, roles in `roles/`. `requirements.yml` remains for dev convenience (`ansible-galaxy collection install -r requirements.yml`).

## Common commands

```bash
# Local test environment: two VMs (debian1 @ 192.168.60.10, rocky1 @ 192.168.60.20).
# `vagrant up` provisions via install.yml against inventories/vagrant/inventory.
vagrant up

# Dev mode from the checkout (ansible.cfg provides vagrant inventory + roles_path)
ansible-playbook playbooks/install.yml

# Playbooks without a fixed host group take the target as an extra var
ansible-playbook playbooks/barman_install.yml -e target_hosts=debian1

# Validate a playbook without running it
ansible-playbook --syntax-check -e target_hosts=database playbooks/<playbook>.yml

# Build and use as an installed collection (FQCN invocation)
ansible-galaxy collection build --force
ansible-galaxy collection install gliatti-devops-1.0.0.tar.gz --force
ansible-playbook gliatti.devops.install -i <inventory>
```

Ansible is not installed on this Windows machine; syntax-check and lint run through Docker (collections land in `/work/.collections`, gitignored):

```bash
# PowerShell (Git Bash mangles the mount path)
docker run --rm -v "C:\Users\robin\Documents\git\github.com\gliatti\ansible:/work" -w /work -e ANSIBLE_CONFIG=/work/ansible.cfg -e ANSIBLE_COLLECTIONS_PATH=/work/.collections willhallonline/ansible:latest ansible-playbook --syntax-check -i inventories/vagrant/inventory -e target_hosts=database playbooks/<playbook>.yml
docker run --rm -v "C:\Users\robin\Documents\git\github.com\gliatti\ansible:/work" -w /work -e ANSIBLE_COLLECTIONS_PATH=/work/.collections willhallonline/ansible:latest ansible-lint --parseable
```

Lint rules live in `.ansible-lint` (excludes `roles/zabbix/`, legacy inventories). There is no test suite or CI; the vagrant environment is the way to verify behavior.

## Architecture

**Thin playbooks, fat roles.** Playbooks in `playbooks/` are small wrappers that call into roles with `include_role` + `tasks_from`, usually targeting the `database` host group. Composite playbooks chain others with `import_playbook` using relative names (e.g. `install.yml` = `postgresql_install.yml` + `ldap2pg_install.yml`) — relative imports and short role names resolve both in dev mode (roles_path) and from the installed collection (verified end-to-end).

**"Manager" task-file convention.** Roles expose entry points named `*_manager.yml` under `tasks/` (e.g. `roles/postgresql/tasks/install_manager.yml`, `conf_manager.yml`, `hba_manager.yml`, `databases_manager.yml`, `roles_manager.yml`, `privileges_manager.yml`). Playbooks pick the one they need via `tasks_from`, so a role is a menu of operations rather than a single monolithic run.

**OS-family dispatch.** Cross-distro roles (postgresql, ldap2pg) branch with `include_tasks: '{{ ansible_os_family }}/install.yml'` into `tasks/Debian/` and `tasks/RedHat/` subdirectories, with RedHat-specific vars in `vars/RedHat.yml`. Both families must be kept working (the vagrant setup has one Debian 12 and one Rocky 9 VM for exactly this).

**Variables.** All role variables are prefixed with the role name (`postgresql_*`, `barman_*`, …) and live in `defaults/main.yml`, which is the reference documentation: defaults are split into "don't override" and "can be overridden" sections, and empty list vars (`postgresql_roles`, `postgresql_privileges`) carry commented examples showing the expected item shape. The PostgreSQL layout convention is `/pgdata`, `/pgwal`, `/pgbackup` with symlinks back into `/etc/postgresql/...`, and memory settings computed from Ansible facts.

## Caveats

- The roles `debian`, `git`, `gpg`, `pki`, `vmware`, `mysecureshell` are minimal implementations created to make their playbooks runnable (the originals lived in the larger GitLab project this repo came from). `vmware` is an explicit placeholder (debug + assert). References to a `postgres` role were remapped to `postgresql`.
- `inventories/vagrant/inventory` carries test groups beyond `database`: `ansible`, `pgbouncer`, `barman_server`, `vmware_manager` (with `vmware_guests: []`) so every playbook targets a real host.
- Package repos on RedHat (PGDG in `roles/postgresql`, Dalibo Labs in `roles/ldap2pg`) are declared via `yum_repository` with GPG keys vendored in each role's `files/` — bootstrap-RPM downloads through the yum module fail on some guests (`[ASN1: NOT_ENOUGH_DATA]` SSL errors). `prerequisites.yml` enables the CRB repo on RedHat (needed for `perl-IPC-Run` pulled by `libpq-devel`).
- On Debian, `postgresql.conf` lives in `/etc/postgresql/<version>/main` (`postgresql_conf_directory`), not in PGDATA like on RedHat, and the `fr_FR.UTF-8` locale must be generated (`conf_manager.yml` does it) or postgres refuses to start with the `lc_*` settings.
- Playbooks that used to take `hosts: "{{ inventory_hostname }}"` / `inventory_host_name` now use the unified `target_hosts` extra var. Standalone playbooks use it strict (with a `# noqa: syntax-check[specific]` for lint) and error without `-e target_hosts=...`; playbooks imported by composite playbooks use `target_hosts | default([])` so the parent lints/parses — without the var their plays match no hosts and are skipped.
- Restart/reload of services goes through role handlers (`roles/postgresql/handlers/`, `roles/pgbouncer/handlers/`); the service name is `postgresql_service_name` (Debian: `postgresql`, RedHat: `postgresql-{{ postgresql_version }}` via `vars/RedHat.yml`).
- The `roles/ldap2pg` Debian/RedHat task files add the Dalibo Labs package repositories (`apt.dalibo.org` / `yum.dalibo.org`) — these are required, ldap2pg is distributed through them.
- The root `hosts` and `offline-hosts` files are legacy inventories for external infrastructure; the vagrant inventory is the one actually wired into the Vagrantfile.
- `ansible.cfg` sets `become_flags=-i` (login shell on privilege escalation) — some tasks depend on the escalated user's profile being loaded.
