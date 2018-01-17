# wpaas
Wordpress as a Service (aka minipages). A fun little project for hosting a bunch of wordpress sites in a bunch of docker containers with sftp access and backups. Perfect if you need a bunch of easily-deployed low traffic WordPress instances without the overhead of VMs or mucking around with php and webservers. Proof of concept-ish and missing some bells and whistles.

## Requirements
* Ansible (and the [geerlingguy.docker role](https://galaxy.ansible.com/geerlingguy/docker/))
* Linux host (tested with Ubuntu 16.04) with working sendmail.
* A wildcard SSL cert for the fqdn of the host.
* A wildcard DNS name for subdomains/dns records for accessing instances.
## How to use
This is a rough and rather brief HowTo for setting up your own minipages instances. Feel free to report any problems, questions or bugs on [GitHub](https://github.com/moegyver/wpaas/).
* Create an inventory with hostnames
```
[minipages]
your.host.tld
```
* Create a `host_vars` file specificing a root password for mysql installations, ssl-certs and the wordpress instances you want (see README in minipages role)
* Run the `minipages.yml` playbook Example: `ansible-playbook -i <path_to_inventory> minipages.yml`
* Go to `https://<instance_name>.your.host.tld` to start setting up your WordPress

## How do I update?
Existing WordPress instances can be updated by running the regular WordPress update from the admin interface. If you want to update the underlying php container you can pull a newer version of the container and rerun the playbook. It should replace the existing container and reuse the existing WordPress installation and database. The same goes for MySQL. I have not tested this though.

## Problems? Questions? Bugs?
This is a fun little weekend project and far from complete or perfect. Feel free to report any problems, questions or bugs on [GitHub](https://github.com/moegyver/wpaas/) anyway, I promise I'll do my best to help.
