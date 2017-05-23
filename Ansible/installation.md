Ansible - Installation pour Debian 8
==
<br/>

Installer rapidement Ansible 2.0 sous Debian 8

#### Pré-requis.
```bash

apt-get install python-pip python-dev build-essential libssl-dev libffi-dev sudo

# Install via python-pip
pip install PyYAML && pip install jinja2 && pip install --upgrade setuptools
pip install paramiko --upgrade && pip install --upgrade pyasn1

```

#### Installation.
```bash

wget http://releases.ansible.com/ansible/ansible-2.0.0.0.tar.gz
tar xzfv ansible-2.0.0.0.tar.gz
cd ansible-2.0.0.0/

make all
make install

```

#### Test
```bash

~/ansible-2.0.0.0# ansible --version                                                                                                                      
ansible 2.0.0.0
  config file =
  configured module search path = Default w/o overrides

~/ansible-2.0.0.0# ansible localhost -m ping                                                                                                              
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

```

#### Installation standard préco par Ansible
```bash
vi /etc/apt/sources.list
deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main

apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
apt-get update
apt-get install ansible

```
