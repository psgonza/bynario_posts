Title: Using different sudo passwords for the same host group in Ansible
Date: 2017-05-31 00:01:17 +0200
Slug: Using-different-sudo-passwords-for-the-same-host-group-in-Ansible
category: posts
tag: bynario

I'll write this down here before I forget.

I have 2 VPSs that I manage with Ansible and recently I updated the root password for one of the servers, and my old playbook, which worked fine if all the hosts in the group have the same password, stopped working.

I tried some stuff that didn't work, until I found out that the way to solve this is described in this link:

[Ansible - Become](http://docs.ansible.com/ansible/become.html)

Basically I added a connection variable called ```ansible_become_pass``` for each of my servers, and assigned them with the variable names inside of my Vault. Like this:

```
[vps]
foo.com ansible_user=foo ansible_ssh_private_key_file=/server/id_rsa_foo ansible_become_pass='{{ foo_sudo_pass }}'
bar.foo.com ansible_user=bar ansible_ssh_private_key_file=/server/id_rsa_bar ansible_become_pass='{{ bar_sudo_pass }}'
```

The variables ```foo_sudo_pass``` and ```bar_sudo_pass``` are the root passwords for each server and they are stored in my Vault file (called passwords.yml).

Those variables will be readed at the time the playbook is executed... So you could have something as simple as this:

```
$ cat upgradeAll.yml
---
- hosts: vps
  become: true
  tasks:
  - name: "apt-get update"
    apt:
      upgrade: dist
      update_cache: yes
      cache_valid_time: 7200
```

And run the playbook with the verbose option to get some more information with this command:

```$ ansible-playbook upgradeAll.yml -e@/server/playbooks/passwords.yml --vault-password-file vault_pass.txt  -v ```

Let's see an example:

```
$ ansible-playbook upgradeAll.yml -e@/server/playbooks/passwords.yml --vault-password-file vault_pass.txt  -v
PLAY [vps] 
*********************************************************************************

TASK [Gathering Facts] 
*********************************************************************************
ok: [foo.com]
ok: [bar.foo.com]

TASK [read vars] 
*********************************************************************************
ok: [foo.com] => {"ansible_facts": {"foo_sudo_pass": "pass1", "bar_sudo_pass": "pass2", "test_sudo_pass": "testpass"}, "changed": false}
ok: [bar.foo.com] => {"ansible_facts": {"foo_sudo_pass": "pass1", "bar_sudo_pass": "pass2", "test_sudo_pass": "testpass"}, "changed": false}

TASK [apt-get update] 
*********************************************************************************
changed: [foo.com] => {"changed": true, "msg": "Reading package lists.
[...]

PLAY RECAP 
*********************************************************************************
foo.com                : ok=3    changed=1    unreachable=0    failed=0
bar.foo.com            : ok=3    changed=1    unreachable=0    failed=0

$
```

It is interesting to notice how I added the extra variables in the command by using ```-e@/server/playbooks/passwords.yml```, otherwise it won't work!

Later!
