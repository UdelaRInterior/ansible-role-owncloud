# Owncloud Ansible role

This role installs and configures an Owncloud instance for a debian/Ubuntu server.

The role's main actions are:
-   [x] Packages dependencies installation.
-   [x] Database configuration (if located on the same host).
-   [x] Strengthened files permissions and ownership following Owncloud recommendations.
-   [x] Web server configuration.
-   [x] Redis Server installation.
-   [x] Strengthened TLS configuration following _Mozilla SSL Configuration Generator_, intermediate profile by default, modern profile available.
-   [x] Post installation of Owncloud applications

This role is an adaptation to Owncloud of [`aalaesar.install_nextcloud`](https://galaxy.ansible.com/aalaesar/install_nextcloud) role.

## Requirements
### Ansible version
Ansible 2.4
### Python libraries
To use `ipwrap` filter in Ansible, you need to install the netaddr Python library on a computer on which you use Ansible (it is not required on remote hosts). It can usually be installed with either your system package manager or using pip:
```bash
$ pip install netaddr
```
### Setup module:
The role uses facts gathered by Ansible on the remote host. If you disable the Setup module in your playbook, the role will not work properly.
### Root access
This role requires root access, so either configure it in your inventory files, run it in a playbook with a global `become: yes` or invoke the role in your playbook like:
> playbook.yml:

```YAML
- hosts: dnsserver
  become: yes
  roles:
    - role: udelarinterior.install_owncloud
```

## Role Variables

Role's variables (and their default values):

### Choose the version

**_WARNING: Since Nexcloud 11 requires php v5.6 or later, command line installation will fail on old OS without php v5.6+ support._**

_Known issue while installing Owncloud 11 on an Ubuntu 14.04 system:_ [#27](https://github.com/aalaesar/install_owncloud/issues/27)

An URL will be generated following naming rules used in the owncloud repository
_Not following this rules correctly may make the role unable to download owncloud._

#### Repository naming rules:
Some variables changes depending on the channel used and if get_latest is true.
This table summarize the possible cases.

|channel|latest|major&latest|major|full|special|
|---|---|---|---|---|---|
|**community**|yes/no|_null_ \|9\|10\|...|_null_|"10.0.3"|_null_|
|**testing**|_null_|_null_|_null_|"11.0.1"|_null_ \|"RC(n)\|beta(n)"|
|**daily**|yes/no|_null_ \|master\|stable9\|...|master\|9\|10\|...|_null_|_null_ \|"YYYY-MM-DD"|

**major&latest** = major value when latest is true
_null_ = "not used"
#### version variables:
```YAML
owncloud_version_channel: "community" # community | testing | daily
```
Specify the main channel to use.
```YAML
owncloud_get_latest: true
```
Specify if the "latest" archive should be downloaded. This paramenter doesn't work for Owncloud yet.

```YAML
# owncloud_version_major: 10
```
Specify what major version you desire. Major version doesn't work for Owncloud yet.

```YAML
owncloud_version_full: "10.3.2"
```
The full version of the desired owncloud instance. type **M.F.P** _(Major.Feature.Patch)_ This works for Owncloud.

```YAML
# owncloud_version_special: ""
```
Specify a special string in the archive's filename.
For prereleases: "RCn|beta" | for daily "YYYY-MM-DD"

```YAML
owncloud_repository: "https://download.owncloud.com/server"
```
Repository's URL.

```YAML
owncloud_archive_format: "zip" # zip | tar.bz2
```
Choose between the 2 archive formats available in the repository.

```YAML
# owncloud_full_url:
```
_If you don't like rules..._
Specify directly a full URL to the archive. The role will skip the url generation and download the archive. **Requires owncloud_version_major to be set along**.
#### Examples:
- Download your own archive:
  (_you **must** specify the owncloud major version along_)
```YAML
owncloud_full_url: https://h2g2.com/42/owncloud.zip
owncloud_version_major: 42
```
-   Choose the latest release (default):
```YAML
owncloud_version_channel: "releases"
owncloud_get_latest: true
```
-   Choose the latest v10 release (Nextcloud only):
```YAML
owncloud_version_channel: "releases"
owncloud_get_latest: true
owncloud_version_major: 10
```
-   Choose a specific release:
```YAML
owncloud_version_channel: "community"
owncloud_get_latest: false
owncloud_full_version: "10.3.2"
```
-   Get the owncloud 11.0.1 prerelease 1:
```YAML
owncloud_version_channel: "testing"
owncloud_version_full: "11.0.1"
owncloud_version_special: "RC1"
```
-   Get the latest daily:
```YAML
owncloud_version_channel: "daily"
owncloud_get_latest: true
```
-   Get the latest daily for stable 10:
```YAML
owncloud_version_channel: "daily"
owncloud_get_latest: true
owncloud_version_major: "stable10"
```
-   Get the daily for master at january 1rst 2017:
```YAML
owncloud_version_channel: "daily"
owncloud_get_latest: false
owncloud_version_major: "master"
owncloud_version_special: "2017-01-01"
```
### Main configuration
```YAML
owncloud_trusted_domain:
  - "{{ ansible_fqdn }}"
  - "{{ ansible_default_ipv4.address }}"
```
The list of domains you will use to access the same Owncloud instance.
```YAML
owncloud_trusted_proxies: []
```
The list of trusted proxies IPs if Owncloud runs through a reverse proxy.
```YAML
owncloud_instance_name: "{{ owncloud_trusted_domain | first }}"
```
The name of the Owncloud instance. By default, the first element in the list of trusted domains
### WebServer configuration
```YAML
owncloud_install_websrv: true
```
The webserver setup can be skipped if you have one installed already.
```YAML
owncloud_websrv: "apache2"
```
The http server used by owncloud. Available values are: **apache2** or **nginx**.
```YAML
owncloud_disable_websrv_default_site: false
```
Disable the default site of the chosen http server. (`000-default.conf` in Apache, `default` in Nginx.)
```YAML
owncloud_websrv_template: "templates/{{owncloud_websrv}}_nc.j2"
```
The jinja2 template creating the instance configuration for your webserver.
You can provide your own through this parameter.
```YAML
owncloud_webroot: "/opt/owncloud"
```
The Owncloud root directory.
```YAML
owncloud_data_dir: "/var/ncdata"
```
The Owncloud data directory. This directory will contain all the Owncloud files. Choose wisely.
```YAML
owncloud_admin_name: "admin"
```
Defines the Owncloud admin's login.
```YAML
owncloud_admin_pwd: "secret"
```
Defines the Owncloud admin's password.  
**Not defined by default**  
If not defined by the user, a random password will be generated.

```YAML
owncloud_max_upload_size: "512m"
```
Defines the max size allowed to be uploaded on the server.  
Use 0 to __disable__.

### Redis Server configuration
```YAML
owncloud_install_redis_server: true
```
Whenever the role should install a redis server on the same host.
```YAML
owncloud_redis_host: '/var/run/redis/redis.sock'
```
The Hostname of redis server. It is set to use UNIX socket as redis is on same host. Set to hostname if it is not the case.
```YAML
owncloud_redis_port: 0
```
The port of redis server. Port 0 is for socket use. Default redis port is 6379.
```YAML
owncloud_redis_settings:
  - { name: 'redis host', value: '"{{ owncloud_redis_host }}"' }
  - { name: 'redis port', value: "{{ owncloud_redis_port }}" }
  - { name: 'memcache.locking', value: '\OC\Memcache\Redis' }
```
Settings to use redis server with Owncloud

### Owncloud Background Jobs
```YAML
owncloud_background_cron: True
```
Set operating system cron for executing Owncloud regular tasks. This method enables the execution of scheduled jobs without the inherent limitations the Web server might have.

### Custom owncloud settings
```YAML
owncloud_config_settings:
  - { name: 'overwrite.cli.url', value: 'https://{{ owncloud_trusted_domain | first }}' }
  - { name: 'memcache.local', value: '\OC\Memcache\APCu' }
  - { name: 'open_basedir', value: '/dev/urandom' }
  - { name: 'mysql.utf8mb4', value: 'true' }
  - { name: 'updater.release.channel', value: 'production' } # production | stable | daily | beta
```
Setting custom Owncloud setting in config.php ( [Config.php Parameters Documentations](https://docs.owncloud.com/server/stable/admin_manual/) )

Default custom settings:
-   **Base URL**: 'https:// {{owncloud_instance_name}}'
-   **Memcache local**: APCu
-   **Mysql Character Set**: utf8mb4
-   **PHP read access to /dev/urandom**: Enabled
-   **Updater Relese Channel:** Production
### Database configuration
```YAML
owncloud_install_db: true
```
Whenever the role should install and configure a database on the same host.
```YAML
owncloud_db_host: "127.0.0.1"
```
The database server's ip/hostname where Owncloud's database is located.
```YAML
owncloud_db_backend: "mysql"
```
Database type used by owncloud.

Supported values are:
-   mysql
-   mariadb
-   pgsql _(PostgreSQL)_

```YAML
owncloud_db_name: "owncloud"
```
The Owncloud instance's database name.
```YAML
owncloud_db_admin: "ncadmin"
```
The Owncloud instance's database user's login
```YAML
owncloud_db_pwd: "secret"
```
The Owncloud instance's database user's password.

**Not defined by default.**

If not defined by the user, a random password will be generated.

### TLS configuration
```YAML
owncloud_install_tls: true
```
TLS setup can be skipped if you manage it separately (e.g. behind a reverse proxy).
```YAML
owncloud_tls_enforce: true
```
Force http to https.
```YAML
owncloud_mozilla_modern_ssl_profile: true
```
Force Mozilla modern SSL profile in webserver configuration (intermediate profile is used when false).
```YAML
owncloud_hsts: false
```
Set HTTP Strict-Transport-Security header (e.g. "max-age=15768000; includeSubDomains; preload").

_(Before enabling HSTS, please read into this topic first)_
```YAML
owncloud_tls_cert_method: "self-signed"
```
Defines various method for retrieving a TLS certificate.
-   **self-signed**: generate a _one year_ self-signed certificate for the trusted domain on the remote host and store it in _/etc/ssl_.
-   **signed**: copy provided signed certificate for the trusted domain to the remote host or in /etc/ssl by default.
  Uses:
```YAML
  # Mandatory:
  owncloud_tls_src_cert: /local/path/to/cert
  # ^local path to the certificate's key.
  owncloud_tls_src_cert_key: /local/path/to/cert/key
  # ^local path to the certificate.

  # Optional:
  owncloud_tls_cert: "/etc/ssl/{{ owncloud_trusted_domain }}.crt"
  # ^remote absolute path to the certificate's key.
  owncloud_tls_cert_key: "/etc/ssl/{{ owncloud_trusted_domain }}.key"
  # ^remote absolute path to the certificate.
```
-   **installed**: if the certificate for the trusted domain is already on the remote host, specify its location.
  Uses:
```YAML
  owncloud_tls_cert: /path/to/cert
  # ^remote absolute path to the certificate's key. mandatory
  owncloud_tls_cert_key: /path/to/cert/key
  # ^remote absolute path to the certificate. mandatory
  owncloud_tls_cert_chain: /path/to/cert/chain
  # ^remote absolute path to the certificate's full chain- used only by apache - Optional
```

### System configuration

install and use a custom version for PHP instead of the default one:
```YAML
php_ver: '7.1'
php_custom: yes
php_ver: "{{ php_ver }}"
php_dir: "/etc/php/{{ php_ver }}"
php_bin: "php-fpm{{ php_ver }}"
php_pkg_apcu: "php-apcu"
php_pkg_spe:
  - "php{{ php_ver }}-imap"
  - "php{{ php_ver }}-imagick"
  - "php{{ php_ver }}-xml"
  - "php{{ php_ver }}-zip"
  - "php{{ php_ver }}-mbstring"
  - "php-redis"
php_socket: "/run/php/{{ php_ver }}-fpm.sock"
```

```YAML
owncloud_websrv_user: "www-data"
```
system user for the http server
```YAML
owncloud_websrv_group: "www-data"
```
system group for the http server
```YAML
owncloud_mysql_root_pwd: "secret"
```
root password for the mysql server

**Not defined by default**

If not defined by the user, and mysql/mariadb is installed during the run, a random password will be generated.

### Generated password
The role uses Ansible's password Lookup:
-   If a password is generated by the role, ansible stores it **locally** in **owncloud_instances/{{ owncloud_trusted_domain }}/** (relative to the working directory)
-   if the file already exist, it reuse its content
-   see [the ansible password lookup documentation](https://docs.ansible.com/ansible/latest/plugins/lookup/password.html) for more info

### Post installation:
#### Applications installation

Since **v1.3.0**, it is possible to download, install and enable owncloud applications during a post-install process.

The application (app) to install have to be declared in the `owncloud_apps` dictionary in a "key:value" pair.
-   The app name is the key
-   The download link, is the value.

```YAML
owncloud_apps:
  app_name_1: "http://download_link.com/some_archive.zip"
  app_name_2: "http://getlink.com/another_archive.zip"
```

Alternatively, if you need to configure an application after enabling it, you can use this structure.
```YAML
owncloud_apps:
  app_name_1:
    source: "http://download_link.com/some_archive.zip"
    conf:
      parameter1: ldap:\/\/ldapsrv
      parameter2: another_value
```

**Notes:**
-   Because the role is using owncloud's occ, it is not possible to install an app from the official owncloud app store.
-   If you know that the app is already installed, you can give an empty string to skip the download.
-   The app name need the be equal to the folder name located in the **apps folder** of the owncloud instance, which is extracted from the downloaded archive.
The name may not be canon some times. (like **appName-x.y.z** instead of **appName**)
-   The role will **not** update an already enabled application.
-   The configuration is applied only when the app in enabled the first time:
Changing a parameter, then running the role again while the app is already enabled will **not** update its configuration.
-   this post_install process is tagged and can be called directly using the `--tags install_apps` option.

## Dependencies

none

## Example Playbook
### Case 1: Installing a quick Owncloud demo
In some case, you may want to deploy quickly many instances of Owncloud on multiple hosts for testing/demo purpose and don't want to tune the role's variables for each hosts: Just run the playbook without any additional variable (all default) !

```YAML
---
- hosts: server
  roles:
   - role: aalaesar.install_owncloud
```

-   This will install a Owncloud 10.0.1 instance in /opt/owncloud using apache2 and mysql.
-   it will be available at **https:// {{ ansible default ipv4 }}**  using a self signed certificate.
-   Generated passwords are stored in **owncloud_instances/{{ owncloud_trusted_domain }}/** from your working directory.

### Case 1.1: specifying the version channel, branch, etc.
You can choose the version channel to download a specific version of owncloud. Here's a variation of the previous case, this time installing the latest nightly in master.
```YAML
---
- hosts: server
  roles:
   - role: aalaesar.install_owncloud
     owncloud_version_channel: "daily"
     owncloud_version_major: "master"
```

### Case 2: Using letsencrypt with this role.
This role is not designed to manage letsencrypt certificates. However you can still use your certificates with owncloud.

You must create first your certificates using a letsencrypt ACME client or an Ansible role like [this one] (https://github.com/jaywink/ansible-letsencrypt)

then call _install_owncloud_ by setting `owncloud_tls_cert_method: "installed"`

Here 2 examples for apache and nginx (because they have slightly different configurations)
```YAML
---
- hosts: apache_server
  roles:
   - role: aalaesar.install_owncloud
     owncloud_trusted_domain:
       - "example.com"
     owncloud_tls_cert_method: "installed"
     owncloud_tls_cert: "/etc/letsencrypt/live/example.com/cert.pem"
     owncloud_tls_cert_key: "/etc/letsencrypt/live/example.com/privkey.pem"
     owncloud_tls_cert_chain: "/etc/letsencrypt/live/example.com/fullchain.pem"

- hosts: nginx_server
  roles:
    - role: aalaesar.install_owncloud
      owncloud_trusted_domain:
        - "example2.com"
      owncloud_tls_cert_method: "installed"
      owncloud_tls_cert: "/etc/letsencrypt/live/example2.com/fullchain.pem"
      owncloud_tls_cert_key: "/etc/letsencrypt/live/example2.com/privkey.pem"
```
### Case 3: integration to an existing system.
-   An Ansible master want to install a new Owncloud instance on an existing Ubuntu 14.04 server with nginx & mariadb installed.
-   As is server do not meet the php requirements for Owncloud 11, he chooses to use the lastest Owncloud 10 release.
-   He wants it to be accessible from internet at _cloud.example.tld_ and from his intranet at _dbox.intra.net_.
-   He already have a valid certificate for the intranet domain in /etc/nginx/certs/ installed
-   he wants the following apps to be installed & enabled : files_external, calendar, agenda, richdocuments (Collabora)
-   The richdocuments app has to be configured to point out to the Collabora domain.

He can run the role with the following variables to install Owncloud accordingly to its existing requirements .

```YAML
---
- hosts: server
  roles:
   - role: aalaesar.install_owncloud
     owncloud_version_major: 10
     owncloud_trusted_domain:
       - "cloud.example.tld"
       - "dbox.intra.net"
     owncloud_websrv: "nginx"
     owncloud_admin_pwd: "secret007"
     owncloud_webroot: "/var/www/owncloud/"
     owncloud_data_dir: "/ncdata"
     owncloud_db_pwd: "secretagency"
     owncloud_tls_cert_method: "installed"
     owncloud_tls_cert: "/etc/nginx/certs/owncloud.crt"
     owncloud_tls_cert_key: "/etc/nginx/certs/owncloud.key"
     owncloud_mysql_root_pwd: "42h2g2"
     owncloud_apps:
       files_external: "" #enable files_external which is already installed in owncloud
       calendar: "https://github.com/owncloud/calendar/releases/download/v1.5.0/calendar.tar.gz"
       contacts: "https://github.com/owncloud/contacts/releases/download/v1.5.3/contacts.tar.gz"
       richdocuments-1.1.25: # the app name is equal to the extracted folder name from the archive
          source: "https://github.com/owncloud/richdocuments/archive/1.1.25.zip"
          conf:
            wopi_url: 'https://office.example.tld'
```

## License

BSD
