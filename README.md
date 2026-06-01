# Ansible Web Tools Installer

Automated Ansible-based installer for a complete self-hosted web toolset. Deploy a private cloud, development infrastructure, or both — all containerized via Docker and served over HTTPS — with a single command.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
  - [Hardware](#hardware)
  - [Software](#software)
  - [Network](#network)
  - [DNS](#dns)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [global_vars.yml](#global_varsyml)
  - [Per-role variables](#per-role-variables)
- [HTTPS Certificates](#https-certificates)
  - [Auto-generated (Let's Encrypt)](#auto-generated-lets-encrypt)
  - [Manual certificates](#manual-certificates)
- [Hosts](#hosts)
- [Playbooks and Scripts](#playbooks-and-scripts)
- [Roles](#roles)
- [Post-Installation Access](#post-installation-access)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Overview

This project installs and configures a curated set of self-hosted web tools on a remote server using Ansible. All services run in Docker containers behind an Nginx reverse proxy with automatic HTTPS via Let's Encrypt (or your own certificates).

Three installer modes are available:

| Installer | Command | What it installs |
|-----------|---------|-----------------|
| Cloud tools | `./install_cloud.sh` | Nextcloud, OnlyOffice, Collabora, e-mail server, OpenLDAP |
| Dev tools | `./install_devtools.sh` | GitLab, Jenkins, Nexus, SonarQube |
| All tools | `./install_all.sh` | Everything above |

---

## Features

### Cloud suite
- **Private cloud storage** via [Nextcloud](https://nextcloud.com/) — file sync, sharing, and Nextcloud Talk for integrated instant messaging and video calls.
- **Online document editing** with MS Office compatibility via [OnlyOffice](https://www.onlyoffice.com/) and LibreOffice compatibility via [Collabora](https://www.collaboraoffice.com/).
- **E-mail server** with SPAM filtering via [Poste.io](https://poste.io/).
- **LDAP directory** for centralized user management via [OpenLDAP](https://github.com/osixia/docker-openldap).

### Development suite
- **Git repositories** via [GitLab](https://gitlab.com/).
- **CI/CD pipelines** via [Jenkins](https://www.jenkins.io/).
- **Artifact repository** via [Nexus Repository](https://www.sonatype.com/products/nexus-repository).
- **Code quality analysis** via [SonarQube](https://www.sonarqube.org/).

### Infrastructure
- **Reverse proxy** via [Nginx](https://www.nginx.com/) — single entry point for all services.
- **Automatic HTTPS** via [Let's Encrypt](https://letsencrypt.org/) with auto-renewal, or bring your own certificates.
- **Fully unattended deployment** via [Ansible](https://www.ansible.com/).
- **Docker-based services** for simple installation, isolation, and future updates.

---

## Requirements

### Hardware

| Profile | RAM | CPU cores | Free disk |
|---------|-----|-----------|-----------|
| Cloud tools only | 4 GB min / 8 GB rec. | 2 min / 4 rec. | 10 GB min |
| Dev tools only | 8 GB min / 16 GB rec. | 4 min / 6 rec. | 15 GB min |
| All tools | 16 GB min | 6 min | 25 GB min |

> **Note:** The more disk space available, the better — cloud storage, e-mail data, artifacts, and VCS repositories all grow over time.

### Software

**On the target server:**
- Debian 10+ or Ubuntu Server LTS
- SSH server running
- `root` user or a user with `sudo` permissions

**On your local machine (where you run the installer):**
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) 2.9+
- The `community.docker` Ansible collection:
  ```bash
  ansible-galaxy collection install community.docker
  ```

### Network

The target server needs outbound internet access. For external access, the following ports must be reachable (open/forwarded if behind NAT):

| Port | Protocol | Service |
|------|----------|---------|
| 80 | HTTP | Nginx (redirect to HTTPS / Let's Encrypt challenge) |
| 443 | HTTPS | Nginx (all web services) |
| 25 | SMTP | E-mail |
| 110 | POP3 | E-mail |
| 143 | IMAP | E-mail |
| 465 | SMTPS | E-mail |
| 587 | SMTP submission | E-mail |
| 993 | IMAPS | E-mail |
| 995 | POP3S | E-mail |
| 636 | LDAPS | OpenLDAP |

### DNS

All subdomain names are configurable via Ansible variables. The defaults below assume `yourdomain.com` as the root domain.

- `yourdomain.com` → A record pointing to your server's public IP
- `cloud.yourdomain.com` → CNAME to root domain *(Nextcloud)*
- `mail.yourdomain.com` → CNAME to root domain *(e-mail server)*
- `ldap.yourdomain.com` → CNAME to root domain *(OpenLDAP)*
- `collabora.yourdomain.com` → CNAME to root domain *(Collabora)*
- `onlyoffice.yourdomain.com` → CNAME to root domain *(OnlyOffice)*
- `gitlab.yourdomain.com` → CNAME to root domain *(GitLab)*
- `jenkins.yourdomain.com` → CNAME to root domain *(Jenkins)*
- `nexus.yourdomain.com` → CNAME to root domain *(Nexus)*
- `sonar.yourdomain.com` → CNAME to root domain *(SonarQube)*
- MX record: `mail.yourdomain.com` with priority 10

---

## Quick Start

1. **Configure DNS** as described [above](#dns).
2. **Forward ports** if the server is behind NAT — see [Network](#network).
3. **Edit `hosts.cfg`** with your server's IP and credentials — see [Hosts](#hosts).
4. **Edit `global_vars.yml`**: set at least `root_domain` and `email_certbot`.
5. **Set service passwords** in the per-role `vars/main.yml` files — see [Per-role variables](#per-role-variables).
6. **Run the installer**:
   ```bash
   ./install_cloud.sh      # cloud tools only
   ./install_devtools.sh   # dev tools only
   ./install_all.sh        # everything
   ```

---

## Configuration

### global_vars.yml

The main configuration file. Located at the project root.

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `public_server` | Use Let's Encrypt for HTTPS and validate DNS. Set to `False` to use [manual certificates](#manual-certificates). | No | `True` |
| `install_proxy_base` | Install the Nginx proxy base. **Must be `True` on first installation.** | No | `True` |
| `install_root` | Create the root domain HTTPS cert and Nginx vhost. | No | `True` |
| `install_nextcloud` | Install Nextcloud. | No | `True` |
| `install_email` | Install e-mail server. | No | `True` |
| `install_ldap` | Install OpenLDAP. | No | `True` |
| `install_collabora` | Install Collabora. | No | `True` |
| `install_onlyoffice` | Install OnlyOffice. | No | `True` |
| `install_gitlab` | Install GitLab. | No | `True` |
| `install_jenkins` | Install Jenkins. | No | `True` |
| `install_nexus` | Install Nexus. | No | `True` |
| `install_sonar` | Install SonarQube. | No | `True` |
| `root_domain` | Your root domain (also used as the e-mail and LDAP domain). | **Yes** | `mydomain.com` |
| `cloud_domain` | Nextcloud subdomain. | No | `cloud.{{root_domain}}` |
| `mail_domain` | E-mail server subdomain. | No | `mail.{{root_domain}}` |
| `ldap_domain` | OpenLDAP subdomain. | No | `ldap.{{root_domain}}` |
| `collabora_domain` | Collabora subdomain. | No | `collabora.{{root_domain}}` |
| `onlyoffice_domain` | OnlyOffice subdomain. | No | `onlyoffice.{{root_domain}}` |
| `gitlab_domain` | GitLab subdomain. | No | `gitlab.{{root_domain}}` |
| `jenkins_domain` | Jenkins subdomain. | No | `jenkins.{{root_domain}}` |
| `nexus_domain` | Nexus subdomain. | No | `nexus.{{root_domain}}` |
| `sonar_domain` | SonarQube subdomain. | No | `sonar.{{root_domain}}` |
| `email_certbot` | E-mail address for Let's Encrypt notifications. | **Yes** (if `public_server: True`) | `admin@mydomain.com` |
| `root_directory_deploy` | Base installation directory on the server. | No | `/opt` |
| `directory_deploy_cloud` | Installation directory for cloud services. | No | `{{root_directory_deploy}}/cloud` |
| `directory_volumes_cloud` | Docker volumes directory for cloud services. | No | `{{directory_deploy_cloud}}/dockervolumes` |
| `directory_scripts_cloud` | Scripts directory for cloud services. | No | `{{directory_deploy_cloud}}/dockerscripts` |
| `directory_deploy_devtools` | Installation directory for dev tools. | No | `{{root_directory_deploy}}/devtools` |
| `directory_volumes_devtools` | Docker volumes directory for dev tools. | No | `{{directory_deploy_devtools}}/dockervolumes` |
| `directory_scripts_devtools` | Scripts directory for dev tools. | No | `{{directory_deploy_devtools}}/dockerscripts` |
| `directory_deploy_proxy` | Installation directory for the proxy. | No | `{{root_directory_deploy}}/proxy` |

### Per-role variables

These files contain secrets that **must** be changed before installation:

#### `roles/openldap/vars/main.yml`

| Variable | Description | Default |
|----------|-------------|---------|
| `initial_openldap_admin_password` | OpenLDAP admin password | `changemenow` |
| `openldap_organisation_name` | Organisation name in the LDAP directory | `organisation` |

#### `roles/collabora/vars/main.yml`

| Variable | Description | Default |
|----------|-------------|---------|
| `secret_collabora` | Collabora admin secret | `longsecretfornoonefigureout` |

#### `roles/onlyoffice/vars/main.yml`

| Variable | Description | Default |
|----------|-------------|---------|
| `secret_onlyoffice` | OnlyOffice JWT secret | `longsecretfornoonefigureout` |

#### `roles/nextcloud/vars/main.yml`

| Variable | Description | Default |
|----------|-------------|---------|
| `initial_nextcloud_admin_password` | Nextcloud admin password | `changemenow` |

#### `roles/checkdocker/vars/main.yml`

| Variable | Description | Default |
|----------|-------------|---------|
| `docker_compose_version` | Docker Compose version to install | `1.25.4` |

---

## HTTPS Certificates

All services are served exclusively over HTTPS.

### Auto-generated (Let's Encrypt)

Set `public_server: True` in `global_vars.yml` and provide a valid `email_certbot`. The installer will:
- Obtain a certificate for each enabled service subdomain.
- Configure auto-renewal automatically.

### Manual certificates

Set `public_server: False` in `global_vars.yml` and place your certificates in the corresponding directories **before** running the installer. Certificates must be named exactly:

| File | Content |
|------|---------|
| `cert.pem` | Public certificate |
| `privkey.pem` | Private key |
| `chain.pem` | CA chain / intermediate certificate |

Certificate directories:

| Directory | Service |
|-----------|---------|
| `roles/proxy/files/root_certs/` | Root domain (`yourdomain.com`) — only needed if `install_root: True` |
| `roles/proxycloud/files/cloud_certs/` | Nextcloud (`cloud.yourdomain.com`) |
| `roles/proxycloud/files/collabora_certs/` | Collabora (`collabora.yourdomain.com`) |
| `roles/proxycloud/files/onlyoffice_certs/` | OnlyOffice (`onlyoffice.yourdomain.com`) |
| `roles/proxycloud/files/mail_certs/` | E-mail server (`mail.yourdomain.com`) |
| `roles/proxycloud/files/ldap_certs/` | OpenLDAP (`ldap.yourdomain.com`) |
| `roles/proxydevtools/files/gitlab_certs/` | GitLab (`gitlab.yourdomain.com`) |
| `roles/proxydevtools/files/jenkins_certs/` | Jenkins (`jenkins.yourdomain.com`) |
| `roles/proxydevtools/files/nexus_certs/` | Nexus (`nexus.yourdomain.com`) |
| `roles/proxydevtools/files/sonar_certs/` | SonarQube (`sonar.yourdomain.com`) |

---

## Hosts

Edit `hosts.cfg` to point the installer at your target server. The file uses standard [Ansible inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html) syntax:

```ini
[default]
10.0.0.12  ansible_ssh_user=myuser  ansible_ssh_pass=mypassword  ansible_become_pass=mypassword
```

- If the user is `root`, `ansible_become_pass` is not required.
- If the user is non-root, it must have `sudo` privileges and `ansible_become_pass` must be set.
- SSH key authentication is also supported — omit `ansible_ssh_pass` and ensure your key is in the server's `authorized_keys`.

---

## Playbooks and Scripts

Each installer script is a thin wrapper that calls `ansible-playbook` against the corresponding playbook:

| Script | Playbook | Installs |
|--------|----------|---------|
| `install_cloud.sh` | `install_cloud.yml` | Cloud services (Nextcloud, OnlyOffice, Collabora, e-mail, OpenLDAP) |
| `install_devtools.sh` | `install_devtools.yml` | Dev tools (GitLab, Jenkins, Nexus, SonarQube) |
| `install_all.sh` | `install_all.yml` | All cloud and dev tool services |

Playbooks are standard [Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html) composed of [roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) executed in sequence.

---

## Roles

| Role | Description | Included in |
|------|-------------|------------|
| `checkdocker` | Installs Docker and dependencies on the target host. | All playbooks |
| `checkdomainscloud` | Validates that cloud subdomains resolve to the server's public IP. Active only when `public_server: True`. | `install_cloud`, `install_all` |
| `checkdomainsdevtools` | Validates that dev tool subdomains resolve to the server's public IP. Active only when `public_server: True`. | `install_devtools`, `install_all` |
| `proxy` | Installs the Nginx reverse proxy base (if `install_proxy_base: True`) and the root HTTPS vhost (if `install_root: True`). | All playbooks |
| `proxycloud` | Configures Nginx vhosts and HTTPS certificates for cloud services. | `install_cloud`, `install_all` |
| `proxydevtools` | Configures Nginx vhosts and HTTPS certificates for dev tool services. | `install_devtools`, `install_all` |
| `email` | Installs the Poste.io e-mail server (if `install_email: True`). | `install_cloud`, `install_all` |
| `collabora` | Installs Collabora Online (if `install_collabora: True`). | `install_cloud`, `install_all` |
| `onlyoffice` | Installs OnlyOffice Document Server (if `install_onlyoffice: True`). | `install_cloud`, `install_all` |
| `nextcloud` | Installs Nextcloud (if `install_nextcloud: True`). | `install_cloud`, `install_all` |
| `openldap` | Installs OpenLDAP (if `install_ldap: True`). | `install_cloud`, `install_all` |
| `gitlab` | Installs GitLab CE (if `install_gitlab: True`). | `install_devtools`, `install_all` |
| `jenkins` | Installs Jenkins (if `install_jenkins: True`). | `install_devtools`, `install_all` |
| `nexus` | Installs Nexus Repository Manager (if `install_nexus: True`). | `install_devtools`, `install_all` |
| `sonar` | Installs SonarQube Community Edition (if `install_sonar: True`). | `install_devtools`, `install_all` |
| `showadminpasswords` | Displays the auto-generated admin passwords for Jenkins and Nexus at the end of the run. | `install_devtools`, `install_all` |

---

## Post-Installation Access

After a successful run, services are available at the following URLs (using default subdomain values):

| Service | URL | Default credentials |
|---------|-----|---------------------|
| Nextcloud | `https://cloud.yourdomain.com` | `admin` / value of `initial_nextcloud_admin_password` |
| E-mail (Poste.io) | `https://mail.yourdomain.com` | Set up on first login |
| OpenLDAP web UI | `https://ldap.yourdomain.com` | `cn=admin,dc=yourdomain,dc=com` / value of `initial_openldap_admin_password` |
| GitLab | `https://gitlab.yourdomain.com` | Set up on first login |
| Jenkins | `https://jenkins.yourdomain.com` | Shown by `showadminpasswords` role |
| Nexus | `https://nexus.yourdomain.com` | Shown by `showadminpasswords` role |
| SonarQube | `https://sonar.yourdomain.com` | `admin` / `admin` *(change immediately)* |
| Collabora | `https://collabora.yourdomain.com` | Configured from within Nextcloud |
| OnlyOffice | `https://onlyoffice.yourdomain.com` | Configured from within Nextcloud |
| OpenLDAP (LDAPS) | `ldaps://ldap.yourdomain.com:636` | — |

> **Note:** Collabora and OnlyOffice do not expose a user-facing UI at their URLs — they are backend document editing services that integrate with Nextcloud.

---

## Troubleshooting

**SSH connection refused on first run**

Ansible requires an accepted SSH host key. Connect manually before running the installer:
```bash
ssh root@your-server-ip
```
Accept the fingerprint, then re-run the installer.

**Let's Encrypt certificate generation fails**

- Ensure all DNS records are fully propagated before running.
- Verify the required ports (80 and 443) are reachable from the internet.
- Check that the `email_certbot` variable is set to a valid e-mail address.

**Domain validation fails (`checkdomainscloud` / `checkdomainsdevtools`)**

The installer checks that each subdomain resolves to the server's public IP. If it does not, the playbook will abort early. Confirm DNS propagation with:
```bash
dig cloud.yourdomain.com
```

**Services not accessible after installation**

Check that the Nginx proxy container is running on the server:
```bash
docker ps | grep nginx
```

---

## Contributing

Pull requests are welcome. For significant changes, please open an issue first to discuss what you would like to change.

---
