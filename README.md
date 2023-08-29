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

Example Running
---------------

```
$ ansible-playbook -i tests/inventory tests/test.yml                                                                            

PLAY [test] ********************************************************************************************************************************************************************************

TASK [/tmp/ansible-role-httpd-ssl : Check if httpd.conf exists] ****************************************************************************************************************************
[WARNING]: Platform openbsd on host 172.23.169.196 is using the discovered Python interpreter at /usr/local/bin/python3.11, but future installation of another Python interpreter could
change the meaning of that path. See https://docs.ansible.com/ansible-core/2.15/reference_appendices/interpreter_discovery.html for more information.
ok: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Copy httpd.conf from template if not exists] ***********************************************************************************************************
changed: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Check if acme-client.conf exists] **********************************************************************************************************************
ok: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Check if etc vhosts directory exist] *******************************************************************************************************************
ok: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Create etc vhosts directory] ***************************************************************************************************************************
changed: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Template vhost files] **********************************************************************************************************************************
changed: [172.23.169.196] => (item={'domain': 'foobar.com', 'alias': ['www.foobar.com', 'v6.foobar.com']})
changed: [172.23.169.196] => (item={'domain': 'foo.com', 'alias': ['www.foo.com']})
changed: [172.23.169.196] => (item={'domain': 'foobar.org', 'alias': ['blog.foo.org']})
changed: [172.23.169.196] => (item={'domain': 'test.org', 'alias': ['www.tests.org']})
changed: [172.23.169.196] => (item={'domain': 'test.net', 'alias': ['www.tests.net']})
changed: [172.23.169.196] => (item={'domain': 'test.com', 'alias': ['www.tests.com']})
changed: [172.23.169.196] => (item={'domain': 'openbsd.org', 'alias': ['www.openbsd.org']})

TASK [/tmp/ansible-role-httpd-ssl : Check if vhosts www directory exist] *******************************************************************************************************************
ok: [172.23.169.196] => (item={'domain': 'foobar.com', 'alias': ['www.foobar.com', 'v6.foobar.com']})
ok: [172.23.169.196] => (item={'domain': 'foo.com', 'alias': ['www.foo.com']})
ok: [172.23.169.196] => (item={'domain': 'foobar.org', 'alias': ['blog.foo.org']})
ok: [172.23.169.196] => (item={'domain': 'test.org', 'alias': ['www.tests.org']})
ok: [172.23.169.196] => (item={'domain': 'test.net', 'alias': ['www.tests.net']})
ok: [172.23.169.196] => (item={'domain': 'test.com', 'alias': ['www.tests.com']})
ok: [172.23.169.196] => (item={'domain': 'openbsd.org', 'alias': ['www.openbsd.org']})

TASK [/tmp/ansible-role-httpd-ssl : Ensure htdocs directories exist] ***********************************************************************************************************************
changed: [172.23.169.196] => (item={'domain': 'foobar.com', 'alias': ['www.foobar.com', 'v6.foobar.com']})
changed: [172.23.169.196] => (item={'domain': 'foo.com', 'alias': ['www.foo.com']})
changed: [172.23.169.196] => (item={'domain': 'foobar.org', 'alias': ['blog.foo.org']})
changed: [172.23.169.196] => (item={'domain': 'test.org', 'alias': ['www.tests.org']})
changed: [172.23.169.196] => (item={'domain': 'test.net', 'alias': ['www.tests.net']})
changed: [172.23.169.196] => (item={'domain': 'test.com', 'alias': ['www.tests.com']})
changed: [172.23.169.196] => (item={'domain': 'openbsd.org', 'alias': ['www.openbsd.org']})

TASK [/tmp/ansible-role-httpd-ssl : Template vhost files] **********************************************************************************************************************************
skipping: [172.23.169.196] => (item={'domain': 'foobar.com', 'alias': ['www.foobar.com', 'v6.foobar.com']}) 
skipping: [172.23.169.196] => (item={'domain': 'foo.com', 'alias': ['www.foo.com']}) 
skipping: [172.23.169.196] => (item={'domain': 'foobar.org', 'alias': ['blog.foo.org']}) 
skipping: [172.23.169.196] => (item={'domain': 'test.org', 'alias': ['www.tests.org']}) 
skipping: [172.23.169.196] => (item={'domain': 'test.net', 'alias': ['www.tests.net']}) 
skipping: [172.23.169.196] => (item={'domain': 'test.com', 'alias': ['www.tests.com']}) 
skipping: [172.23.169.196] => (item={'domain': 'openbsd.org', 'alias': ['www.openbsd.org']}) 
skipping: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Add vhost includes to httpd.conf] **********************************************************************************************************************
changed: [172.23.169.196] => (item={'domain': 'foobar.com', 'alias': ['www.foobar.com', 'v6.foobar.com']})
changed: [172.23.169.196] => (item={'domain': 'foo.com', 'alias': ['www.foo.com']})
changed: [172.23.169.196] => (item={'domain': 'foobar.org', 'alias': ['blog.foo.org']})
changed: [172.23.169.196] => (item={'domain': 'test.org', 'alias': ['www.tests.org']})
changed: [172.23.169.196] => (item={'domain': 'test.net', 'alias': ['www.tests.net']})
changed: [172.23.169.196] => (item={'domain': 'test.com', 'alias': ['www.tests.com']})
changed: [172.23.169.196] => (item={'domain': 'openbsd.org', 'alias': ['www.openbsd.org']})

TASK [/tmp/ansible-role-httpd-ssl : Copy acme-client.conf if not exists] *******************************************************************************************************************
changed: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Add acme-client configuration] *************************************************************************************************************************
changed: [172.23.169.196]

TASK [/tmp/ansible-role-httpd-ssl : Reload service httpd] **********************************************************************************************************************************
changed: [172.23.169.196]

PLAY RECAP *********************************************************************************************************************************************************************************
172.23.169.196             : ok=12   changed=8    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
```

After a successfully ran, you will see httpd(8) running on the server only in http://, after running "acme-client" on
each domain you added you should be able to uncomment the https:// chunk on the vhost config and reload the service.

License
-------

BSD

Author Information
------------------

https://x61.ar/
