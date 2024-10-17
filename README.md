# Simple Phoenix deployment with Ansible

By "simple" I mean:

- A single server
- No containers
- Mix releases built locally
- Postgres on the same server
- Some downtime is acceptable

## Getting started

Ensure that Ansible is installed on your system and set up SSH access to the target host.

### Add host

Specify the target host for the deployment by updating the `hosts` file.

```ini
[hosts]
123.123.123.123
```

### Update vars file

Update the variables in `group_vars/hosts/vars` for your environment.

```yaml
# Change me!
user: david
port: 4000
project_name: my_app
# etc...
```

### Create a vault

Ansible Vault is used to encrypt sensitive data, including database credentials and secret keys.

To create a new vault, run the following command:

```bash
ansible-vault create group_vars/hosts/vault
```

To avoid typing the vault password every time you run a task, store the password in a file:

```bash
echo 'my_vault_password' > .vault_pass
```

This file is not tracked by version control.

The vault can be edited by running:

```bash
ansible-vault edit group_vars/hosts/vault
```

Your vault should contain the following secrets:

```yaml
vault_db_user: secret
vault_db_password: secret
vault_secret_key_base: secret
```

### Create user

The `01-create-user.yml` playbook should be run first and can only be run once. It creates a new user that you defined in `group_vars/hosts/vars`) and disables root access via SSH.

## Setup server

```
ansible-playbook 02-configure-server.yml
```

This will install updates, configure [sensible server security](https://www.redhat.com/sysadmin/ansible-linux-server-security), setup Postgres and create the database with appropriate permissions, and install Caddy.

## Deploy your Phoenix application

Now that the server is configured, all that is left to do is deploy your app:

```
ansible-playbook 03-deploy-application.yml
```

Ansible will build a Mix Release locally and move the tarball to the server. Any modifications to the Caddyfile or service unit file are also applied. Expect 5-10 seconds of downtime. During this time, Caddy will serve a simple HTML page.
