Ansible role for a httpd(8) with SSL
====================================

Ansible role to create a web server with httpd(8) on OpenBSD (>=6.1) and let's encrypt.

Requirements
------------

OpenBSD >=6.1 -{release,stable,current}

Notes
-----

To add the domains you need to modify the file "defaults/main.yml" and adapt it to
what you want, if you have already a /etc/httpd.conf the playbook will just add the
includes of the vhosts added (BE CAREFUL BARELY TESTED :) otherwise it will copy the
httpd.conf in the templates directory and also add at the end the includes.

You need to run this after the playbook to create the certificates, remember to have the
dns entry for the added domains.

```
# acme-client -vAD foobar.com
```

To renew the certs a cronjob must be placed:

```
#!/bin/sh
## debug
#set -x

UPDATE=0

for domain in $(awk '/^domain/ { print $2 }' /etc/acme-client.conf)
do
	acme-client $domain
	ocspcheck -vNo /etc/ssl/$domain.{ocsp,crt}
	if [ $? -eq 0 ]; then
		UPDATE=1
	fi
done
```

We asume that, you already have an entry on pf.conf like:

```
...
pass in on $ext_if proto tcp from any to any port 80
...
pass in on $ext_if proto tcp from any to any port 443
...
```

Or if your web server is behind the firewall, you need something like:

```
...
pass in on $ext_if proto tcp from any to any port 80 \
	rdr-to 10.0.0.10 port 80
pass in on $ext_if proto tcp from any to any port 443 \
	rdr-to 10.0.0.10 port 443
...
```

Example Playbook
----------------

```
---
- hosts: test
  gather_facts: false
  roles:
    - /home/gonzalo/src/ansible-role-httpd
  become: yes
  become_method: doas
```

License
-------

BSD

Author Information
------------------

https://x61.ar/
