# Postgres DB

## Role installation

### create requirements file

```bash
cd ANSIBLE_FOLDER
mkdir requirements
touch requirements/psql.yml
```

### file content

```yaml
---
# ansible-galaxy install -p vss_galaxy_roles --force -r requirements/psql.yml
- src: "https://github.com/virsas/mod-ansible-rpm-psql"
  scm: git
  version: v1.2.0
  name: psql
  path: vss_galaxy_roles
```

## Install role

### Installation

```bash
ansible-galaxy install -p galaxy_roles --force -r requirements/psql.yml
```

### Best practices

If you are using git for your playbooks and sites configuration, add vss_galaxy_roles to your .gitignore file. You are free to download the role directly to your roles directory, but it will be then pushed to your repo too.

## Role access

### Direct access

```bash
$ cd vss_galaxy_roles
$ ls
psql
$ cd psql
$ ls -1
defaults
handlers
LICENSE
meta
README.md
tasks
templates
$
```

### Access from ansible (ansible.cfg)

```bash
[defaults]
gathering = smart
roles_path=./roles:../roles:../../roles:./vss_galaxy_roles
log_path=./ansible_run.log
retry_files_enabled=False
[ssh_connection]
scp_if_ssh=True
host_key_checking=False
```

## Files

### Playbook (./playbooks/psql.yml)

```yaml
---
- hosts: psql
  remote_user: ec2-user
  become: yes
  roles:
    - psql
```

### Inventory (./sites/NAME/inventory)

```txt
[psql]
psql_01
```

### Host vars (./sites/NAME/host_vars/psql_01.yml)

```yaml
---
ansible_ssh_host: 1.1.1.1
```

### Group vars (./sites/NAME/group_vars/psql.yml)

Please see the variables below to get the complete list of possible modifications. The YAML file below is just a basic example of a minimal configuration.

```yaml
---
```

## Usage

```bash
ansible-playbook -i sites/NAME/inventory playbooks/psql.yml --diff
```

## Variables

```yml
---
POSTGRES_CONFIG_TEMPLATE: postgresql.conf.j2
POSTGRES_HBA_TEMPLATE: ph_hba.conf.j2

# Files must exist in ansible/files directory, you can change their names by modifying below values
POSTGRES_SSL_CRT: server.crt
POSTGRES_SSL_KEY: server.key
POSTGRES_SSL_CA: root.crt

POSTGRES_PATH: "/var/lib"

POSTGRES_CONFIG_HOST: "*"
POSTGRES_CONFIG_PORT: "5432"
POSTGRES_CONFIG_MAX_CON: 100
POSTGRES_DATABASES: [{ db: db1, encoding: UTF-8, users: [
          {
            name: user1,
            pass: "userPass",
            # individual database level privileges.
            global_privileges: [
                {
                  # database or schema
                  type: database,
                  # database or schema name
                  objects: db_name,
                  # privileges USAGE, CREATE, etc
                  privileges: "CREATE",
                  grant_option: False,
                },
              ],
            # individual schema level privileges.
            privileges: [
                {
                  # supported types table, partition table, sequence, function or procedure
                  type: table,
                  # ALL_IN_SCHEMA or list of objects the privileges should be applied to like a list of tables in case of table type.
                  objects: ALL_IN_SCHEMA,
                  # privileges, it can be comma separated list like USAGE,SELECT or ALL
                  privileges: "ALL",
                  # name of the schema
                  schema: "public",
                  # Wheather role may grant privileges to others
                  grant_option: False,
                },
                # another example to grant usage, select on all sequences in public schema to user
                {
                  type: sequence,
                  objects: ALL_IN_SCHEMA,
                  privileges: "USAGE,SELECT",
                  schema: "public",
                  grant_option: False,
                },
              ],
          },
        ] }, { db: db2, encoding: UTF-8, users: [{ name: user2, pass: "userPass", privileges: [{ type: table, objects: ALL_IN_SCHEMA, privileges: "ALL", schema: "public", grant_option: False }] }] }]
```

## Optional Variables

```yml
---
# If you specify POSTGRES_STORAGE, the device will be formatted to ext4 and mounted to /var/lib/postgres for data persistancy
POSTGRES_STORAGE: "/dev/sdb"
POSTGRES_PATH: "/var/lib/postgres"

# If you want to override the default configuration, just change the name of any or both config files
POSTGRES_CONFIG_TEMPLATE: postgresql.j2
POSTGRES_HBA_TEMPLATE: ph_hba.j2
# and then place them in ansible/templates directory
```
