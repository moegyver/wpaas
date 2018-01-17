# pr_minipages - Wordpress as a Service
## What does it do?
`pr_minipages` allows you to start as many Wordpress instances as you want on a host with a few simple lines of yaml. This includes SFTP, MySQL access, backups and an SSL proxy that answers on `name`.`inventory_hostname`.
## HowTo
Add some sites and a wildcard SSL certificate and key for `*.{{inventory_hostname}}` to the `host_vars` like this:
```
---
sites:
  - name: fashionpolice       # The name used for the URL
    id: 1                     # The id is for assigning ports
    username: antje           # Username for SFTP and MySQL
    password: judgesfashion   # Password
  - name: allthethings
    id: 2
    username: filip
    password: knowshistools
  - name: beardblog
    id: 3
    username: alejandro
    password: barbudo
ssl_cert: |
  <certificate for *.{{inventory_hostname}}>
ssl_key: |
  <corresponding key>
```
And then run the playbook `minipages`
### How to access the DB
MySQL is available on port `3000 + id`, username and password are as specified in the inventory.
### How to access the files
SFTP is available on port `2000 + id`, username and password are as specified in the inventory.
### How to access wordpress
`https://{{name}}.{{inventory_hostname}}/`, t.  ex. https://allthethings.minipages.bln.int.planetromeo.com/
## Prerequisites
* DNS for the sites is not handled by this role. Make sure that the corresponding FQDNs resolve to the IP of the host machine (for example by setting up a wildcard DNS record for `*.{{inventory_hostname}}`).
* You also need a wildcard certificate and the corresponding key for `*.{{inventory_hostname}}`
## Bonus features
### Site-specific URL and cert
You can add `url`, `ssl_cert` and `ssl_key` to a site definition which creates a proxy for https://url/ with the certificate and key.
```
sites:
  - name: fashionpolice
    id: 1
    username: antje
    password: judgesfashion
    url: fashionpolice.io
    ssl_cert: |
      <cert for fashionpolice.io>
    ssl_key: |
      <corresponding key for the cert>
[...]
ssl_cert: |
  <certificate for *.{{inventory_hostname}}>
ssl_key: |
  <corresponding key>
```
## How does it work?
The setup consists of a nginx proxy that forwards requests to docker-based wordpress installations.
### Proxy
The proxy serves static content and forwards requests for php scripts based on the domain name in the request to the corresponding containers.
### Docker containers
Each site is turned into three docker containers:
* `db-{{name}}` for the MySQL database. (Based on  https://hub.docker.com/_/mariadb/)
* `web-{{name}}` for Wordpress. A new copy of wordpress is present in the container by default (Based on the fpm alpine version of https://hub.docker.com/_/wordpress/).
* `sftp-{{name}}` for SFTP access (Based on the alpine version of https://hub.docker.com/r/atmoz/sftp/).

The state is saved in `/minipages/{htdocs,db}/{{name}}` and backups are in `/minipages/backup/` for htdocs and `/minipages/backup/{{name}}/` for database backups.
## Known issues and limitations
* The proxy needs a wildcard DNS name or manual DNS names for every site.
* SFTP and MySQL usernames can't be changed.
* No support for multiple MySQL and SFTP users.
* The minipages only supports Wordpress atm. It is however easily extendable.
* Restoring a site from the backups involves manually restoring the DB and files.
* Deleting minipages with the corresponding playbook `remove-minipages` doesn't remove it from the inventory.
* Monitoring for the containers, the proxy, the ssl-certs and the backup cronjobs is missing so far.
* `id` > 59 will end up running backups concurrently and `id` > 1000 may result in port clashes.
