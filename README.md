# Sensible Phoenix deployments using Ansible

This playbook configures a Fedora server with sensible defaults and easy deployment of your Phoenix app.

It does not use containers or provide zero downtime deployment (although downtime is kept to a minimum), and Postgres is located on the same server as the web app.

## Getting started

- Ensure that Ansible is installed on your system.
- Set up SSH access to the target hosts.

### 1. Add host

Specify the target host for the deployment by updating the `hosts` file:

```ini
[hosts]
123.123.123.123
```

### 2. Set variables

Update the variables in `group_vars/hosts/vars` to match your specific deployment needs.

Ansible Vault is used to encrypt sensitive data, including database credentials and the Phoenix secret key.

To create a new vault, run the following command:

```bash
ansible-vault create group_vars/hosts/vault
```

This will open an editor where you can add your secrets.

To avoid typing the vault password every time you run a task, store the password in a file:

```bash
echo 'my_vault_password' > .vault_pass
```

This file is not tracked by version control. Double check `.gitignore` to be sure.

Add secrets to the vault:

```bash
ansible-vault edit group_vars/hosts/vault
```

The following variables are required:

```yaml
db_user: secret
db_password: secret
secret_key_base: secret
```

### 3. Create user

The `01-create-user.yml` playbook should be run first and can only be run once. It creates a new user (defined in `group_vars/hosts/vars`) and disables root access via SSH.
