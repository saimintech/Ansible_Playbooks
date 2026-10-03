# Ansible_Playbooks

Ansible playbooks for setting up a single Ubuntu 22.04 server: CloudPanel, MySQL, an Apache reverse proxy with Let's Encrypt for a Node.js app, and an Apache vhost serving a browser based PHP file editor.

All playbooks target `localhost` with a local connection, so you run them on the server you are configuring.

## Playbooks

| Playbook | What it does | Extra vars |
|----------|--------------|------------|
| `cloudpanel/install_cloudpanel.yml` | Updates and dist-upgrades packages, downloads the CloudPanel CE v2 installer, checks it against a pinned SHA-256, then runs it | none |
| `mysql/install_mysql.yml` | Installs `mysql-server` and PyMySQL, sets the root password (`mysql_native_password`), removes anonymous users and the `test` database | `mysql_root_password` |
| `reverse_proxy/apache_reverse_proxy.yml` | Installs Apache, Certbot, Node.js 18 (NodeSource), latest npm and PM2; creates an HTTP vhost, gets a certificate with the webroot method, then switches to an HTTPS vhost that proxies `/` to `127.0.0.1:<local_port>` and redirects HTTP to HTTPS | `domain_name`, `local_port` |
| `editor/editor.yml` | Creates `/var/www/<domain>`, enables an HTTP vhost, copies `editor.php` and `editor.config.php` into the webroot, gets a certificate, then switches to an HTTPS vhost | `domain_name` |

```
reverse_proxy:   client --443--> Apache (Let's Encrypt) --http--> 127.0.0.1:<local_port> (your Node app, e.g. under PM2)
editor:          client --443--> Apache --> /var/www/<domain>/editor.php
```

## Included tools

- `editor/editor.php`: a single file PHP code editor (simon-thorpe/editor, 2016 build). It can browse, edit and download files and, by default, run shell commands as the web server user. Its password is read from `editor.config.php`.
- `mysql/adminer.php`: Adminer 4.8.1, a single file database admin UI. No playbook deploys it; copy it to a webroot by hand if you want it.

**Before using either tool:** set a strong password in `editor/editor.config.php` (the committed value is only a placeholder), restrict access by IP or HTTP auth at the Apache level, and update Adminer to a current release. The editor's `$ALLOW_SHELL` flag hides the shell UI but does not gate every command path, so treat a logged in editor session as shell access to the server. Adminer gives full database access to anyone who can log in.

## Prerequisites

```bash
sudo apt update && sudo apt -y upgrade
sudo apt -y install curl wget git ansible
```

The full `ansible` package includes the `community.mysql` collection used by the MySQL playbook. For the Apache playbooks, the domain's DNS must already point at the server and ports 80/443 must be open, or Certbot will fail.

## Usage

Run from the repository root:

```bash
# CloudPanel (fresh server only)
ansible-playbook -i hosts.ini cloudpanel/install_cloudpanel.yml

# MySQL
ansible-playbook -i hosts.ini mysql/install_mysql.yml -e "mysql_root_password=<password>"

# Apache reverse proxy to a local Node app
ansible-playbook -i hosts.ini reverse_proxy/apache_reverse_proxy.yml -e "domain_name=example.com local_port=3000"

# PHP editor vhost
ansible-playbook -i hosts.ini editor/editor.yml -e "domain_name=example.com"
```

`hosts.ini` only defines `localhost` with `ansible_connection=local`.

## Notes

- CloudPanel installs its own web and database stack and expects a clean server. Do not combine it with the Apache or MySQL playbooks on the same host.
- If CloudPanel publishes a new installer, the checksum task fails until the hash in the playbook is updated.
- Passing `mysql_root_password` with `-e` leaves it in shell history. A vars file (`-e @vars.yml`) or Ansible Vault avoids that.
- Certbot is registered with `admin@<domain_name>`.

## Configuration

Extra vars: `domain_name`, `local_port`, `mysql_root_password`. No environment variables are used.

## Author

Built by Saim Safdar - https://saim.me
