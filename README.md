Ansible role for a httpd(8) with SSL
====================================

Ansible role to create a web server with httpd(8) on OpenBSD (>=6.1) and let's encrypt.

Requirements
------------

OpenBSD >=6.1 -{release,stable,current}

Notes
-----

You need to run this after the playbook:

```
# acme-client -vAD foobar.com
```

To renew the certs a cronjob must be placed:

```
#!/bin/sh
acme-client foobar.com

if [ $? -eq 0 ]
then
	/etc/rc.d/httpd reload
fi
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

And also you already have a DNS entry for your domain.


Example Playbook
----------------

```
---
- hosts: test
   roles:
     - role: gonzalo-.httpd-ssl
   become: yes
   become_method: doas

   vars:
    domain: 'foobar.com'
    alias: 'www.foobar.com'
    httpd_conf: '/etc/httpd'
    www_dir: '/var/www/sites'
```

License
-------

BSD

Author Information
------------------

https://x61.sh/
