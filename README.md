
A set of Ansible scripts to setup your personal mail server (and more) for your home...

## Introduction
This project has been created for those who simply want to host their emails - and more - at home,
and don't want to manage the full installation process manually, from scratch.

It is made to be unobtrusive, standard compliant, secure, robust, extensible and automatic

- Unobtrusive: Most of the packages are coming from the official Debian repository. The others are coming from a maintained repository. *No git clone* here...Once installed, use ad update it normally like a Debian, with apt.
- Standard compliant: For instance, the system not only generate the DKIM records, it publish them for you on Gandi DNS server!
- Secure: The LDAP server is setup to store passwords encrypted. Password policies and default password policy are setup as well. The distribution can be updated with normal apt update/upgrade.
- Robust: the DNS records update script is very safe, and you can run it in test mode. In this mode, the new zone version will be created, but not activated. If there is no change to your DNS record, the new version will be deleted.
- Extensible: By using LDAP for user authentications, you can use other software, like nextcloud, gitlab or even an OpenVPN server, without having to remember more passwords.
- Automatic: Most tasks are automated. Even the external IP address detection and DNS update process. In theory, you could use this with a dynamic IP address.

## Current status

| Feature                                                                                                          | Status      | 
| ---------------------------------------------------------------------------------------------------------------- | ----------- | 
| LDAP users database, SSL & TLS certificates, password policies, integration with PAM                             | Done        | 
| SSL Certificates creation using letsencrypt, backup and publication on Gandi                                     | Done        | 
| DKIM keys generation and backup, publication on Gandi                                                            | Done        | 
| SPF records generation and publication on Gandi                                                                  | Done        | 
| DMARC record generation and publication on Gandi (report generation planned)                                     | Done        | 
| Generation and publication of automatic Thunderbird configuration (MS Outlook planned)                           | Done        | 
| Postfix configuration and installation, with LDAP lookups, SSL & TLS, DKIM and Antispam (rspamd)                 | Done        | 
| Dovecot configuration, IMAP, POP, Quotas, ManageSieve, Spam and ham autolearn, Sieve auto answers                | Done        | 
| Roundcube webmail, https, sieve filters access, password change, automatic identity creation                     | Done        | 
| AppArmor securisation for nginx, dovecot, postfix, etc                                                           | In progress | 
| Automatic migration from old mail server                                                                         | Planned     | 
| Automatic encrypted off-site backup                                                                              | Planned     | 
| Web proxy with privacy and parent filtering features                                                             | Planned     | 

## Folders:
- config: Ansible hosts file Configuration.
- preseed: Ansible scripts to create an automatic ISO image installer for testing or live system.
- install: Ansible scripts to install the whole server environment.
- backup: Automatic backup of certificates and DKIM keys.

## Mail server installation

First, check that you can login to your server, using SSH on the root account:

`ssh root@mail.example.com`

Replace mail.example.com with the address of your mail server.
This will also add the server to your known_hosts file

### Copy the example files to create your basic setup

Run the following code to create your custom files

```
  cd install/playbook/variables
  cp accounts.example.yml accounts.yml
  cp common.example.yml common.yml
  cp dovecot.example.yml dovecot.yml
  cp postfix.example.yml postfix.yml
  cp webmail.example.yml webmail.yml
```

File contents
- common.yml: common variables and basic network configuration (domain)
- accounts.yml: User and group accounts information
- dovecot.yml: Dovecot mail server configuration
- postfix.yml: Postfix MTA configuration
- webmail.yml: Webmail (roundcube) configuration

This process will be simplified in a future version to reduce the number of files to duplicate.

### Run the Ansible scripts to setup your email server
The installation folder is using Ansible to setup the mail server
For instance, inside the install folder, run the following command:

`ansible-playbook -vv -i ../config/hosts.yml playbooks/install.yml`

The certificates are generated using LetsEncrypt service, with one for each service. Examples for example.com domain:

  - openldap: ldap.example.com
  - postfix: smtp.example.com
  - dovecot: imap.example.com
  - the webmail: webmail.example.com
  
The generated certificates and DKIM keys will be automatically saved on your local computer, into the backup folder. This folder is ignored by git. If you restart the installation from scratch using a new server, these certificates and DKIM keys will be used, so you do not end up requesting more certificates or updating your DNS server more than necessary. This is particularly useful in development phase.

## What do you need in production

### Basic requirements

- A low consumption hardware box to plug on your router, I personally use [pondesk](https://www.pondesk.com/) boxes.
- A computer to run the Ansible scripts.
- A static IP address from your ISP (Not mandatory, but works better)
- Some basic newtork knowledge

For automatic DNS records creation and update:
- A registered domain name, with [Gandi](https://gandi.net/)

### Automatic update of the DNS entries
A custom script automatically register domains entries to Gandi, if you provide an API key.
This is useful so you do not have to manage the entries yourself, or if even if you don't have a static IP address.

Here what to do to obtain an API key:

1. Visit the [API key page on Gandi](https://www.gandi.net/admin/api_key)
2. Activate of the API on the test platform.
3. Activate of the production API

Your API key will be activated on the test platform.

__Notes:__

- The initial creation of DNS records for certificate generation should take some time.
- DNS automatic update is actually limited to Gandi, but it should be easy to add more.
- Once the script has been run, the backup folder contains your certificates and DKIM public keys. If you are rebuilding your server from scratch, the same certificates and keys will be used.
- This is a work in progress and a project I am maintaining on my spare time. Although I am trying to be very careful, there might be some errors. In this case, just fill a bug report, or take part.
- I am privileging stability over features. The master branch should stay stable for production.
- There are other similar projects on internet and especially github you could check, for instance [Sovereign](https://github.com/sovereign/sovereign) which offer more features and more up to date packages, outside the Debian official repositories.

__TODO__:

I am planning to test / try / add the following features, in *almost* no particular order:

- Automatic LUKS setup for the ISO image installer.
- Add a caldav / carddav server (Any that works with LDAP authentication)
- Tor, torrent download station, etc...
- Add a jabber server (Any that works with LDAP authentication)
- Add optional components (e.g. [Gogs](https://gogs.io/), [openvpn](https://openvpn.net/), [Syncthing](https://syncthing.net/), etc)
- Test other mail systems, like Cyrus, Sogo, etc.

Any ideas welcome...
