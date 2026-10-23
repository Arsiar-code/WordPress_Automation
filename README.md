# WordPress Automation with Ansible

Ansible playbooks that set up a standard WordPress website on an Ubuntu server: Apache, MySQL, PHP and WordPress, fully automated.

The setup follows the official Ubuntu tutorial [Install and configure WordPress](https://ubuntu.com/tutorials/install-and-configure-wordpress), with each manual step turned into an Ansible task. This project was created as my IPA (Individuelle praktische Arbeit) for my Informatiker EFZ Plattformentwicklung apprenticeship.

## What it does

| Playbook | Purpose |
|---|---|
| `dependencies.yaml` | Installs Apache, MySQL, PHP with the required extensions, and PyMySQL |
| `wordpress_setup.yaml` | Creates `/srv/www` and downloads and unpacks the latest WordPress |
| `mysql_setup.yaml` | Creates the database and user, creates `wp-config.php` from the sample, and inserts the database credentials |
| `apache_setup.yaml` | Creates the Apache site from a Jinja2 template, removes the default page, and enables the WordPress site |

Other files:

- `hosts.ini`: inventory (the target server is in the group `IPA`)
- `vars.yaml`: variables
- `secrets.yaml`: sensitive values such as the database password, encrypted with **Ansible Vault**
- `wordpress.conf.j2`: Jinja2 template for the Apache virtual host

## Requirements

- A control machine with Ansible installed
- A target server running Ubuntu with SSH access and sudo rights
- The `community.mysql` Ansible collection:

```bash
ansible-galaxy collection install community.mysql
```

## Usage

1. Add your server to `hosts.ini` under the group `IPA`.
2. Create your own encrypted secrets file. It must define `my_db_password`:

   ```bash
   ansible-vault create secrets.yaml
   ```

3. Put `wordpress.conf.j2` in a `templates/` folder (the Apache playbook looks for `templates/wordpress.conf.j2`) and adjust the `ServerName` to your domain.
4. Run the playbooks in this order:

   ```bash
   ansible-playbook -i hosts.ini dependencies.yaml --ask-vault-pass
   ansible-playbook -i hosts.ini wordpress_setup.yaml --ask-vault-pass
   ansible-playbook -i hosts.ini mysql_setup.yaml --ask-vault-pass
   ansible-playbook -i hosts.ini apache_setup.yaml --ask-vault-pass
   ```

5. Open the server's address in a browser and finish the WordPress setup in the web interface.

## Notes and possible improvements

- Move hardcoded values (server name, database name and user) into `vars.yaml`.
- Generate the WordPress security keys and salts automatically from the WordPress API.
- Check the `creates:` and `removes:` paths of the `a2ensite` and `a2dissite` tasks so the Apache tasks stay idempotent.
- Combine the playbooks into a single `site.yaml` that imports them in order.

## Security

Passwords are never stored in plain text. `secrets.yaml` is encrypted with Ansible Vault, and the task that writes the database password to `wp-config.php` runs with `no_log: true`.
