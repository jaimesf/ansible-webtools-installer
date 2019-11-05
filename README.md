# Ansible Web Tools Installer

You can install easily the basic service for a little startup, that doesn't need a big infrastructure to work. 
Change vars to your needs, the target host, and just execute 
```
./install_cloud.sh
```
You will have a private cloud, instant messaging, online document editor and an e-mail server.

Additionaly, if you need an CI/CD environment for software development startup, your command is
```
./install_devtools.sh
```
This give you Gitlab, Jenkins, Nexus installations automatically.

Or if you need all of them, the magic words are:
```
./install_all.sh
```

## Features at a Glance

* Private cloud to store all you need via [Nextcloud](https://nextcloud.com/)
* Instant messaging and videoconference without 3rd party solutions.
* Online document editor with MS Office ([OnlyOffice](https://www.onlyoffice.com/es/)) and Libreoffice ([Collabora](https://www.collaboraoffice.com/)) compat.
* E-mail server to manage your domain, with SPAM filter included, via [Poste.io](https://poste.io/)
* [Gitlab](https://gitlab.com/) as your main VCS.
* [Jenkins](https://jenkins.io/) as CI/CD manager.
* [Nexus](https://www.sonatype.com/product-nexus-repository) as your main artifact repository.
* [Nginx](https://www.nginx.com/) as your main channel to serve all data.
* Auto generation https certs, via [Let's Encrypt](https://letsencrypt.org/)
* Use your own https certs instead generated
* [Ansible](https://www.ansible.com/) install all your services on unattended way.
* All services, deployed via [Docker](https://www.docker.com/) to simplify the installations and update processes.

# Requirements

## Hardware
- **Cloud tools**
    * _RAM_: 4 GB minimum. 8 GB recommended.
    * _CPU cores_: 2 cores minimum. 4 recommended
    * _Storage free space_: 10 GB minimum. **Try to get all free space you can: Your storage space on the cloud, and e-mail data, depends on this.**
- **Development tools**
    * _RAM_: 8 GB minimum. 16 GB recommended.
    * _CPU cores_: 4 cores minimum. 6 recommended
    * _Storage free space_: 15 GB minimum. **Try to get all free space you can: Your artifact repository data, VCS repositories and compilations, depends on this.**
- **All web tools**
    * _RAM_: 16 GB minimum.
    * _CPU cores_: 6 cores minimum.
    * _Storage free space_: 25 GB minimum. **Try to get all free space you can: All your data, depends on this.**
## Software
- **Supported OS**
    * _Debian_ (10 recommended)
    * _Ubuntu server_ (LTS recommended)
- **Access**
    * _SSH server_ installed
    * _Root user_ or user with _sudo permissions_

## Network
- Access to Internet required from server.
- Access from Internet if the server has to be accessed from outside.
    * If the server, it's a server with public IP on it's network interface, that's all you need.
    * If the server is behind NAT, you need to forward this ports:
        + 80 (HTTP)
        + 443 (HTTPS)
        + 25 (SMTP)
        + 110 (POP3)
        + 143 (IMAP)
        + 465 (SMTP)
        + 587 (SMTP)
        + 993 (IMAP)
        + 995 (POP3)

## DNS
All names for the subdomains could be customize via ansible vars at the installation time.

- Root domain pointing to your public IP with A registry. 
- cloud.yourdomain.com as CNAME registry to your root domain (If [Nextcloud](https://nextcloud.com/) will be installed)
- mail.yourdomain.com as CNAME registry to your root domain (If [E-mail server](https://poste.io/) will be installed)
- collabora.yourdomain.com as CNAME registry to your root domain (If [Collabora](https://www.collaboraoffice.com/) will be installed)
- onlyoffice.yourdomain.com as CNAME registry to your root domain (If [OnlyOffice](https://www.onlyoffice.com/es/) will be installed)
- gitlab.yourdomain.com as CNAME registry to your root domain (If [Gitlab](https://gitlab.com/) will be installed)
- jenkins.yourdomain.com as CNAME registry to your root domain (If [Jenkins](https://jenkins.io/) will be installed)
- nexus.yourdomain.com as CNAME registry to your root domain (If [Nexus](https://www.sonatype.com/product-nexus-repository) will be installed)
- MX register pointing to mail.yourdomain.com with priority 10.

# Variables
Vars are distributed among multiple files on the repository.
### global_vars.yml
| Name | Description | Change required | Default Value  |
|------|-------------| --------------- | -------------  |
| **public_server** | The server is public and use letsencrypt to generate certs, and check domains. **False** value required to put your [certs on their own directories](#manual-certificates) | No | True |
| **install_proxy_base** |Create proxy base. **_Mandatory to True if there is no previous installation_**. | No | True |
| **install_root** | Create root domain with https cert. Usefull if you deploy some other service on root directory on the future. | No | True |
| **install_nextcloud** | Install nextcloud server. | No | True |
| **install_email** | Install email server. | No | True |
| **install_collabora** | Install collabora server. | No | True |
| **install_onlyoffice** |Install onlyoffice server. | No | True |
| **install_gitlab** | Install gitlab server. | No | True |
| **install_jenkins** | Install jenkins server. | No | True |
| **install_nexus** | Install nexus server. | No | True |
| **root_domain** | Your root domain (And email domain name). | **Yes** | mydomain.com |
| **cloud_domain** | Subdomain for nextcloud installation. | No | cloud.{{root_domain}} |
| **mail_domain** | Subdomain for email server installation. | No | mail.{{root_domain}} |
| **collabora_domain** | Subdomain for collabora server installation. | No. | collabora.{{root_domain}} |
| **onlyoffice_domain** | Subdomain for onlyoffice server installation. | No | onlyoffice.{{root_domain}} |
| **gitlab_domain** | Subdomain for gitlab installation. | No | gitlab.{{root_domain}} |
| **jenkins_domain** | Subdomain for jenkins installation. | No | jenkins.{{root_domain}} |
| **nexus_domain** | Subdomain for nexus installation. | No | nexus.{{root_domain}} |
| **email_certbot** | Email to use on letsencrypt cert generator. | **Yes** | admin@mydomain.com |
| **root_directory_deploy** | Root installation directory. | No | /opt |
| **directory_deploy_cloud** | Installation directory cloud services. | No | {{root_directory_deploy}}/cloud |
| **directory_volumes_cloud** | Installation directory for data cloud services. | No | {{directory_deploy_cloud}}/dockervolumes |
| **directory_scripts_cloud** | Installation directory for scripts cloud services. | No | {{directory_deploy_cloud}}/dockerscripts |
| **directory_deploy_devtools** |Installation directory devtools services. | No | {{root_directory_deploy}}/devtools |
| **directory_volumes_devtools** |Installation directory for data devtools services. | No | {{directory_deploy_devtools}}/dockervolumes |
| **directory_scripts_devtools** | Installation directory for scripts devtools services. | No | {{directory_deploy_devtools}}/dockerscripts |
| **directory_deploy_proxy** | Installation directory proxy. | No | {{root_directory_deploy}}/proxy |

### roles/checkdocker/vars/main.yml
| Name | Description | Change required | Default Value  |
|------|-------------| --------------- | -------------  |
| **docker_compose_version** | Docker compose version to install | No | 1.24.1 |

### roles/collabora/vars/main.yml
| Name | Description | Change required | Default Value  |
|------|-------------| --------------- | -------------  |
| **secret_collabora** | Secret for Collabora admin user | **Yes** | longsecretfornoonefigureout |

### roles/onlyoffice/vars/main.yml
| Name | Description | Change required | Default Value  |
|------|-------------| --------------- | -------------  |
| **secret_onlyoffice** | Secret for Onlyoffice | **Yes** | longsecretfornoonefigureout |

### roles/nextcloud/vars/main.yml
| Name | Description | Change required | Default Value  |
|------|-------------| --------------- | -------------  |
| **initial_nextcloud_admin_password** | Password for Nextcloud admin user | **Yes** | changemenow |

# HTTPS certificates

All services will be deployed over HTTPS, and certificates are needed.

### Auto generated
If your server will have access from Internet, you can use the free https certs generator [Let's Encrypt](https://letsencrypt.org/), setting up **public_server** var to **True**. This option generate https certs for all services subdomains to install, and you don't pay attention to renovate it.

### Manual certificates
If your server won't be accessed from Internet, or if it's public and you have certs for all subdomains, you can put your certs on their directories, and these will be installed.

Your certs have to be named as:
* Certificate: **cert.pem**
* Private key: **privkey.pem**
* CA cert: **chain.pem**
 
Directories to save your certs:
* **roles/proxy/files/root_certs** for root certs, to you main domain {{root_domain}} (Only if you set up **install_root** var to _True_)
* **roles/proxycloud/files/cloud_certs** for Nextcloud certs, to default subdomain cloud.{{root_domain}}
* **roles/proxycloud/files/collabora_certs** for Collabora certs, to default subdomain collabora.{{root_domain}}
* **roles/proxycloud/files/onlyoffice_certs** for Onlyoffice certs, to default subdomain onlyoffice.{{root_domain}}
* **roles/proxycloud/files/mail_certs** for e-mail certs, to default subdomain mail.{{root_domain}}
* **roles/proxydevtools/files/gitlab_certs** for Gitlab certs, to default subdomain gitlab.{{root_domain}}
* **roles/proxydevtools/files/jenkins_certs** for Jenkins certs, to default subdomain jenkins.{{root_domain}}
* **roles/proxydevtools/files/nexus_certs** for Nexus certs, to default subdomain nexus.{{root_domain}}

# Playbooks and installers

[Ansible playbook](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html) is a list of independent complex tasks (Named as [roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)), to execute consecutively to reach you target.

Three playbooks exist:
* **install_cloud.yml**: Define list of roles to install all cloud services (E-mail, Nextcloud, Collabora, Onlyoffice).
* **install_devtools.yml**: Define list of roles to install all development services (Gitlab, Jenkins, Nexus).
* **install_all.yml**: Define list of roles to install all cloud and development services.

Related to these playbook, exists three script to launch easily the playbooks:
* **install_cloud.sh**: Install cloud services via **install_cloud.yml** playbook.
* **install_devtools.sh**: Install devtools services via **install_cloud.yml** playbook.
* **install_all.sh**: Install all services via **install_all.yml** playbook.

# Roles
List of roles and their description
### roles/nextcloud/vars/main.yml
| Name | Description | Included on playbook  |
|------|-------------| --------------------  |
| **checkdocker** | Check and install docker and it's dependencies on the target host. | All |
| **checkdomainscloud** | Check Nextcloud, e-mail, collabora and onlyoffice subdomains with their public IPs (Only with **public server** var active). | install_all, install_cloud |
| **checkdomainsdevtools** | Check Gitlab, Jenkins and Neux subdomains with their public IPs(Only with **public server** var active). | install_all, install_devtools |
| **proxy** | Install proxy (Only with **install_proxy_base** var active) and the root certificate (Only with **install_root** var active). | All |
| **proxycloud** | Install proxy config, generate certs https (Only with **public_server** var active), copy your own https cert certificates (Only with **public_server** var to False) to Nextcloud, E-mail, Collabora and Onlyoffice | install_all, install_cloud |
| **proxydevtools** | Install proxy config, generate certs https (Only with **public_server** var active), copy your own https cert certificates (Only with **public_server** var to False) to Gitlab, Jenkins and Nexus | install_all, install_devtools |
| **email** | Install e-mail server (Only with **install_email** var to True) | install_all, install_cloud |
| **collabora** | Install Collabora server (Only with **install_collabora** var to True)  | install_all, install_cloud |
| **onlyoffice** | Install Onlyoffice server (Only with **install_onlyoffice** var to True)  | install_all, install_cloud |
| **nextcloud** |Install Nextcloud server (Only with **install_nextcloud** var to True)  | install_all, install_cloud |
| **gitlab** | Install Gitlab server (Only with **install_gitlab** var to True)  | install_all, install_devtools |
| **jenkins** | Install Jenkins server (Only with **install_jenkins** var to True)  | install_all, install_devtools |
| **nexus** | Install Nexus server (Only with **install_nexus** var to True)  | install_all, install_devtools |
| **showadminpasswords** | Show initial admin passwords auto generated by Jenkins and Nexus | install_all, install_devtools |

# Hosts

To execute installer, you will need modify hosts.cfg to enter your access data.

Example: 
```
10.0.0.12      ansible_ssh_user=myuser  ansible_ssh_pass=mypassword ansible_become_pass=mypassword
```

If your user, it's not the _root_ user, you need sudo permissions to execute this installer and ansible_become_pass is required.


# Installation

First of all, _you need to install Ansible on your machine (Not the server, you own computer)_ to execute installer. Ansible can be installed on many ways, described on it's [website](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

Steps to install:

1. Set up [DNS config](#dns)
2. [Forward ports](#network) to your server if needed.
3. Set up you target host on [hosts.cfg](#hosts).
4. Set up **root_domain** var on [global-vars.yml](#global_varsyml)
5. Setup for https certs
    1. Auto generated certs: Set up **email_certbot** var on [global-vars.yml](#global_varsyml) and **public_server** to True
    2. Own certs: Copy all [your certs properly named to their directories](#manual-certificates)
6. Set up other vars, as _admin passwords_.
7. Execute the installer you need:
    1. **./install_cloud.sh** to install all cloud services.
    2. **./install_devtools.sh** to install all dev tools services.
    3. **./install_all.sh** to install all services
8. If you installed Jenkins and Nexus, last role, show you admin passwords.

After the installation process you can access services on this URIs if you didn't change subdomains
* **Nextloud**: https://cloud.yourdomain.com
* **E-mail**: https://mail.yourdomain.com
* **Collabora**: https://collabora.yourdomain.com doesn't show you nothing. This URI is for configure collabora on Nextcloud.
* **Onlyoffice**: https://onlyoffice.yourdomain.com doesn't show you nothing relevant. This URI is for configure Onlyoffice on Nextcloud.
* **Gitlab**: https://gitlab.yourdomain.com
* **Jenkins**: https://jenkins.yourdomain.com
* **Nexus**: https://nexus.yourdomain.com

### Problems on installation

* If you can't connect to your server at execute playbooks, first log in to your server manually and accept ssh cert. Ex:`ssh root@10.0.0.15`

---

Pull requests are wellcome ;)



