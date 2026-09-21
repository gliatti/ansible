# gliatti.devops

Collection Ansible pour provisionner des serveurs de bases de données **PostgreSQL**
et l'ensemble de leur écosystème — Barman (sauvegarde), pgBouncer (pooling de
connexions), ldap2pg (synchronisation des rôles depuis un annuaire LDAP) — ainsi que
les rôles système qui les accompagnent (SSH, cron, logrotate, durcissement Debian,
PKI…).

La collection est **multi-OS** et maintenue sur deux familles de distributions :
**Debian 12 (Bookworm)** et **Rocky Linux 9**. Testée aussi sur Debian 13 et
Rocky Linux 10 (au 2026-09-21, Dalibo Labs ne publie pas encore `ldap2pg` pour
Debian 13 : `ldap2pg_install.yml` y échoue au niveau du paquet).

| | |
|---|---|
| **Namespace** | `gliatti.devops` |
| **Version** | `1.0.0` |
| **Ansible requis** | `>= 2.15.0` |
| **Licence** | MIT |
| **Dépôt** | https://github.com/gliatti/ansible |

---

## 📦 Installation

### Depuis le dépôt Git

```bash
ansible-galaxy collection install git+https://github.com/gliatti/ansible.git
```

### Depuis un build local

Depuis un checkout du dépôt :

```bash
ansible-galaxy collection build
ansible-galaxy collection install gliatti-devops-1.0.0.tar.gz
```

### Dépendances

Les collections dépendantes sont déclarées dans `galaxy.yml` et **installées
automatiquement** par `ansible-galaxy` :

| Collection | Version minimale |
|---|---|
| `community.general` | `>= 8.0.0` |
| `community.postgresql` | `>= 3.0.0` |

---

## 🚀 Utilisation

### Une fois la collection installée

Les playbooks de la collection sont appelés par leur nom pleinement qualifié
(FQCN) :

```bash
# Installation complète (PostgreSQL + ldap2pg)
ansible-playbook gliatti.devops.install -i <inventaire>

# Un composant précis
ansible-playbook gliatti.devops.postgresql_install -i <inventaire>
ansible-playbook gliatti.devops.pgbouncer_install -i <inventaire>
```

### En développement, depuis le checkout du dépôt

`ansible.cfg` fournit l'inventaire vagrant par défaut et le `roles_path`, il suffit
donc d'exécuter directement les fichiers du dossier `playbooks/` :

```bash
# Installation complète sur le groupe « database » de l'inventaire vagrant
ansible-playbook playbooks/install.yml

# Playbook sans groupe fixe : cible passée en variable supplémentaire
ansible-playbook playbooks/barman_install.yml -e target_hosts=debian1
```

> **Convention `target_hosts`** — les playbooks qui ne visent pas un groupe
> d'inventaire figé (`database`, `pgbouncer`, `barman_server`…) prennent leur cible
> via la variable supplémentaire `-e target_hosts=<hôte|groupe>`. Sans elle, leur
> jeu (*play*) ne correspond à aucun hôte et est simplement ignoré.

---

## 🧩 Rôles

Tous les rôles préfixent leurs variables par leur nom (`postgresql_*`, `barman_*`…)
et documentent celles-ci dans leur fichier `defaults/main.yml`.

| Rôle | Description | OS supportés |
|---|---|---|
| `postgresql` | Installation et configuration d'un serveur PostgreSQL : cluster, `postgresql.conf`, `pg_hba.conf`, bases, rôles, privilèges, extensions. | Debian, RedHat |
| `barman` | Sauvegarde/PITR via Barman : installation, configuration serveur & côté PostgreSQL, slots de réplication. | Debian, RedHat |
| `pgbouncer` | Pooling de connexions PostgreSQL : installation et déploiement de la configuration. | Debian, RedHat |
| `ldap2pg` | Synchronisation des rôles PostgreSQL depuis un annuaire LDAP (dépôts Dalibo Labs). | Debian, RedHat |
| `ssh` | Déploiement des configurations SSH utilisateur. | Debian, RedHat |
| `cron` | Gestion des tâches cron. | Debian, RedHat |
| `logrotate` | Gestion de la rotation des journaux. | Debian, RedHat |
| `common` | Tâches transverses réutilisables (paquets, services, `lineinfile`, copie de fichiers). | Debian, RedHat |
| `ansible` | Utilitaires Ansible : gestion du fichier `hosts`, prise en compte des VM récemment créées. | Debian, RedHat |
| `debian` | Durcissement et administration Debian (utilisateurs SFTP, `sources.list`, sécurité). *Implémentation minimale.* | Debian |
| `git` | Installation et déploiement de Git. *Implémentation minimale.* | Debian, RedHat |
| `gpg` | Installation et déploiement de GnuPG. *Implémentation minimale.* | Debian, RedHat |
| `pki` | Déploiement d'éléments PKI. *Implémentation minimale.* | Debian, RedHat |
| `mysecureshell` | Déploiement de MySecureShell (SFTP restreint). *Implémentation minimale.* | Debian |
| `vmware` | Provisionnement de machines virtuelles. *Placeholder explicite (debug + assert).* | — |
| `zabbix` | Supervision Zabbix. *Rôle tiers embarqué (vendored) — non maintenu ici.* | Debian, RedHat |

> Les rôles marqués **implémentation minimale** ont été recréés pour rendre leurs
> playbooks exécutables ; leur logique métier d'origine vivait dans un projet plus
> large. Le rôle `vmware` est un **placeholder** (il ne fait que journaliser et
> vérifier ses entrées). Le rôle `zabbix` est **embarqué** tel quel et n'est pas
> maintenu dans ce dépôt.

---

## 📖 Playbooks

Les playbooks vivent dans `playbooks/`. Ceux dont l'entrée « Cible » indique
`target_hosts` doivent recevoir `-e target_hosts=<hôte|groupe>`.

### Composés

| Playbook | Rôle joué | Cible |
|---|---|---|
| `install.yml` | Installation complète : `postgresql_install.yml` + `ldap2pg_install.yml`. | `database` |
| `deploy_databases_wrapper.yml` | Chaîne `deploy_pki.yml` + `deploy_databases.yml`. | (voir sous-playbooks) |
| `deploy_virtual_machines_wrapper.yml` | Provisionnement de VM. | `vmware_manager` |

### PostgreSQL

| Playbook | Usage | Cible |
|---|---|---|
| `postgresql_install.yml` | Installe et initialise le cluster PostgreSQL. | `database` |
| `deploy_postgresql_conf.yml` | Déploie `postgresql.conf`. | `database` |
| `deploy_postgresql_hba.yml` | Déploie `pg_hba.conf`. | `database` |
| `deploy_databases.yml` | Crée bases, rôles et privilèges. | `database` |
| `deploy_sauvegarde_bdd.yml` | Déploie les scripts de sauvegarde des bases. | `target_hosts` |

### Barman

| Playbook | Usage | Cible |
|---|---|---|
| `barman_install.yml` | Installe Barman. | `target_hosts` |
| `barman_configure_server.yml` | Configure le serveur Barman. | `barman_server` |
| `barman_configure_postgres.yml` | Configure PostgreSQL pour Barman. | `target_hosts` |
| `barman_create_slot.yml` | Crée un slot de réplication. | `target_hosts` |

### pgBouncer

| Playbook | Usage | Cible |
|---|---|---|
| `pgbouncer_install.yml` | Installe pgBouncer. | `pgbouncer` |
| `pgbouncer_deploy.yml` | Déploie la configuration pgBouncer. | `pgbouncer` |

### ldap2pg

| Playbook | Usage | Cible |
|---|---|---|
| `ldap2pg_install.yml` | Installe ldap2pg (dépôts Dalibo Labs). | `database` |

### Rôles système & déploiements divers

| Playbook | Usage | Cible |
|---|---|---|
| `deploy_ssh_user_configs.yml` | Configurations SSH utilisateur. | `target_hosts` |
| `deploy_cron.yml` | Tâches cron. | `all` |
| `deploy_logrotate.yml` | Rotation des journaux. | `target_hosts` |
| `deploy_debian_users.yml` | Utilisateurs Debian. | `target_hosts` |
| `deploy_debian_directories.yml` | Arborescence de répertoires Debian. | `target_hosts` |
| `debian_addsftpuser.yml` | Ajout d'un utilisateur SFTP. | `target_hosts` |
| `debian_security_hardening.yml` | Durcissement sécurité Debian. | `all` |
| `debian_sources_list_hardening.yml` | Durcissement du `sources.list`. | `all` |
| `sources_list.yml` | Déploiement du `sources.list`. | `target_hosts` |
| `install_git.yml` / `deploy_git.yml` | Installation / déploiement de Git. | `target_hosts` |
| `install_gpg.yml` / `deploy_gpg.yml` | Installation / déploiement de GnuPG. | `target_hosts` |
| `deploy_pki.yml` | Déploiement PKI. | `ansible` |
| `deploy_mysecureshell.yml` | Déploiement de MySecureShell. | `target_hosts` |
| `deploy_ansible_inventories.yml` | Déploiement des inventaires Ansible. | `ansible` |
| `deploy_virtual_machines.yml` | Provisionnement de VM. | `vmware_manager` |
| `gather_facts.yml` | Collecte des facts. | `all` |
| `example_form.yml` | Exemple de formulaire (`ansible.builtin.pause`). | `target_hosts` |

---

## 🏗️ Conventions du dépôt

- **Playbooks fins, rôles épais.** Les playbooks sont de minces enveloppes qui
  appellent les rôles via `include_role` + `tasks_from`. Les playbooks composés
  enchaînent d'autres playbooks avec `import_playbook`.
- **Convention `*_manager.yml`.** Chaque rôle expose ses points d'entrée sous forme
  de fichiers `tasks/*_manager.yml` (par ex. `install_manager.yml`, `conf_manager.yml`,
  `hba_manager.yml`, `databases_manager.yml`, `roles_manager.yml`,
  `privileges_manager.yml`). Un rôle se comporte donc comme un **menu d'opérations**,
  et le playbook choisit l'entrée voulue via `tasks_from`.
- **Dispatch par famille d'OS.** Les rôles multi-distributions branchent avec
  `include_tasks: '{{ ansible_facts["os_family"] }}/install.yml'` vers des sous-dossiers
  `tasks/Debian/` et `tasks/RedHat/`, les variables spécifiques vivant dans
  `vars/RedHat.yml`.
- **Variables préfixées.** Toutes les variables de rôle sont préfixées par le nom du
  rôle et documentées dans `defaults/main.yml`, qui sert de référence (sections
  « à ne pas surcharger » / « surchargeables », exemples commentés pour les listes
  vides comme `postgresql_roles` ou `postgresql_privileges`).
- **Layout PostgreSQL.** Répartition sur `/pgdata`, `/pgwal`, `/pgbackup` (créés par
  `tasks/layout.yml`, lien `$PGDATA/backups` utilisé par `archive_command`),
  `postgresql.conf` laissé là où la distribution le place (`/etc/postgresql/<version>/main`
  sur Debian, PGDATA sur RedHat), `pg_hba.conf` dans PGDATA, et paramètres mémoire
  (`shared_buffers`, `effective_cache_size`, `work_mem`…) calculés à partir des facts
  Ansible.
- **Handlers.** Les redémarrages et rechargements de services passent par les
  handlers des rôles (`roles/postgresql/handlers/`, `roles/pgbouncer/handlers/`).

---

## 🧪 Environnement de test

Un environnement Vagrant à deux VM permet de valider le comportement sur les deux
familles d'OS simultanément :

| VM | OS | Adresse IP |
|---|---|---|
| `debian1` | Debian 12 | `192.168.60.10` |
| `rocky1` | Rocky Linux 9 | `192.168.60.20` |

```bash
# Provisionne les deux VM via playbooks/install.yml contre l'inventaire vagrant
vagrant up
```

### Validation statique

Ansible n'est pas installé sur le poste de développement (Windows) ; la vérification
de syntaxe et le lint passent par Docker (image `willhallonline/ansible:latest`) :

```bash
# Vérification de syntaxe d'un playbook (sans l'exécuter)
docker run --rm -v "C:\Users\robin\Documents\git\github.com\gliatti\ansible:/work" -w /work \
  -e ANSIBLE_CONFIG=/work/ansible.cfg -e ANSIBLE_COLLECTIONS_PATH=/work/.collections \
  willhallonline/ansible:latest \
  ansible-playbook --syntax-check -i inventories/vagrant/inventory -e target_hosts=database playbooks/install.yml

# Lint de l'ensemble du dépôt
docker run --rm -v "C:\Users\robin\Documents\git\github.com\gliatti\ansible:/work" -w /work \
  -e ANSIBLE_COLLECTIONS_PATH=/work/.collections \
  willhallonline/ansible:latest \
  ansible-lint --parseable
```

Les règles de lint sont définies dans `.ansible-lint`.

---

## 📄 Licence

Distribué sous licence **MIT**.
