# readme

Fri Jan 30 15:27:40 PST 2026

ansible installation of basic stuff

## system

both machines running FreeBSD 15.0-RELEASE

### bastion

* a jump host with ansible installed
* contains ssh-key to access remote FreeBSD host (fbsdhost)

### fbsdhost

* a server that will eventually become a jails host 

---

## pre-requisites

### bastion

* verify ansible install (already done)

    $ pkg info | grep python
    python3-3_4                    Meta-port for the Python interpreter 3.x
    python311-3.11.14              Interpreted object-oriented programming language

    $ pkg info | grep ansible
    py311-ansible-11.7.0           Radically simple IT automation
    py311-ansible-core-2.18.7      Radically simple IT automation


* install ansible freebsd libs

    $ ansible-galaxy collection install vbotka.freebsd

    Starting galaxy collection install process
    Process install dependency map
    Starting collection install process
    Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/vbotka-freebsd-0.8.1.tar.gz to /home/user/.ansible/tmp/ansible-local-2211292k0cw8v/tmppzg37evy/vbotka-freebsd-0.8.1-9vwglgr9
    Installing 'vbotka.freebsd:0.8.1' to '/home/user/.ansible/collections/ansible_collections/vbotka/freebsd'
    vbotka.freebsd:0.8.1 was installed successfully
    'ansible.posix:1.6.2' is already installed, skipping.
    'ansible.utils:5.1.2' is already installed, skipping.
    'community.general:10.7.1' is already installed, skipping.


### fbsdhost

#### manually configured the initial user from root

* replace user 'toddg' with your user on your system(s):
 
    user: user
    in group wheel
    wheel enabled in visudoers


#### install python3 on fbsdhost

    ansible --ask-vault-password -m raw -a "pkg install -y python3" freebsd_hosts

### verify that we have 'ansible_become_pass' for each server

* replace user 'toddg' with your user
* create a new vault file where the key matches what you are using in the hosts file

    $ ansible --ask-vault-password -m debug -a 'var=hostvars[inventory_hostname]' freebsd_hosts

    Vault password:
    fbsdhost | SUCCESS => {
        "hostvars[inventory_hostname]": {
            "ansible_become": "yes",
            "ansible_become_method": "sudo",
            "ansible_become_pass": "..elided..", <------------- present ! 
            "ansible_check_mode": false,
            "ansible_config_file": "servers/ansible/ansible.cfg",
            "ansible_connection": "ssh",
            "ansible_diff_mode": false,
            "ansible_facts": {},
            "ansible_forks": 5,
            "ansible_inventory_sources": [
                "servers/ansible/hosts"
            ],
            "ansible_playbook_python": "/usr/local/bin/python3.11",
            "ansible_python_interpreter": "/usr/local/bin/python3",
            "ansible_ssh_private_key_file": "/home/toddg/.ssh/id_ed25519",
            "ansible_user": "toddg",
            "ansible_verbosity": 0,
            "ansible_version": {
                "full": "2.18.7",
                "major": 2,
                "minor": 18,
                "revision": 7,
                "string": "2.18.7"
            },
            "foo": "bar",
            "group_names": [
                "freebsd_hosts"
            ],
            "groups": {
                "all": [
                    "fbsdhost",
                ],
                "freebsd_hosts": [
                    "fbsdhost",
                ],
                "ungrouped": []
            },
            "inventory_dir": "servers/ansible",
            "inventory_file": "servers/ansible/hosts",
            "inventory_hostname": "fbsdhost",
            "inventory_hostname_short": "fbsdhost",
            "omit": "__omit_place_holder__328d6d5e2f99f1269d20c798d628cc25fb97d41b",
            "playbook_dir": "servers/ansible",
            "vault_fbsdhost_toddg_password": "..elided.." <------------- present ! 
        }
    }


## install basic stuff

    $ ansible-playbook --ask-vault-password playbooks/install_basic_deps.yaml
    Vault password:

    PLAY [freebsd_hosts] *****************************************************************************************************************************************************

    TASK [Gathering Facts] ***************************************************************************************************************************************************
    ok: [fbsdhost]

    TASK [update system packages] ********************************************************************************************************************************************
    ok: [fbsdhost]

    TASK [install required software] *****************************************************************************************************************************************
    ok: [fbsdhost]

    PLAY RECAP ***************************************************************************************************************************************************************
    fbsdhost                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


## install pf firewall

### install deps

* https://galaxy.ansible.com/ui/repo/published/vbotka/freebsd/content/role/pf/

    $ ansible-galaxy role install vbotka.freebsd_pf

    $ ansible-galaxy collection install ansible.posix &&

    ansible-galaxy collection install community.general
    Starting galaxy collection install process
    Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using `--force`.
    Starting galaxy collection install process
    Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using `--force`.

### ...

find the demo configurations

    $ find . | grep pf.conf./freebsd/roles/pf_2_8_0/templates/server-pf.conf.j2

    ./freebsd/roles/pf_2_8_0/templates/router2-pf.conf.j2
    ./freebsd/roles/pf_2_8_0/templates/router3-pf.conf.j2
    ./freebsd/roles/pf_2_8_0/templates/server2-pf.conf.j2
    ./freebsd/roles/pf_2_8_0/templates/default2-pf.conf.j2
    ./freebsd/roles/pf_2_8_0/templates/router1-pf.conf.j2
    ./freebsd/roles/pf_2_8_0/templates/default-pf.conf.j2


** This is the point at which I'm a bit stuck. I'm not sure how to use this ansible plugin **

## links

* https://galaxy.ansible.com/ui/repo/published/vbotka/freebsd/content/role/pf/
* https://www.siberoloji.com/automating-deployments-with-ansible-on-freebsd/
* https://www.digitalocean.com/community/tutorials/how-to-use-vault-to-protect-sensitive-ansible-data
* https://cloudspinx.com/install-prometheus-with-node-exporter-and-grafana-on-freebsd/
* https://www.siberoloji.com/how-to-implement-full-stack-monitoring-with-prometheusgrafana-on-freebsd-operating-system/
* https://www.siberoloji.com/creating-and-managing-freebsd-jails/
* https://www.siberoloji.com/automating-deployments-with-ansible-on-freebsd/
* https://ansible-collection-freebsd.readthedocs.io/en/latest/ug_plugins.html
* https://ansible-collection-freebsd.readthedocs.io/en/latest/ug_module_service.html
* https://galaxy.ansible.com/ui/repo/published/vbotka/freebsd/content/module/service/
* https://ansible-collection-freebsd.readthedocs.io/en/latest/ug_roles.html
* https://galaxy.ansible.com/ui/repo/published/vbotka/freebsd/content/role/pf/
* https://galaxy.ansible.com/ui/repo/published/vbotka/freebsd/content/role/pf/
* https://ansible-collection-freebsd.readthedocs.io/en/latest/ug_module_service.html
* https://blog.andreev.it/2014/06/freebsd-10-and-pf-firewall/
* https://docs.ansible.com/projects/ansible/latest/collections_guide/collections_installing.html
* https://github.com/vbotka/ansible-freebsd-pf?tab=readme-ov-file
* https://docs.freebsd.org/en/books/handbook/firewalls/#firewalls-pf
* https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html

