# ansible-playbooks
Ansible Playbooks

## Vagrant

```bash
$ vagrant init ubuntu/bionic64
$ vagrant up
```

### apply latest version

```bash
$ vagrant box update
```

### log into vm

```bash
$ vagrant ssh
```

### show ssh configurations

```bash
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2200
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile .vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL

# log into vm by pure SSH
$ ssh -i .vagrant/machines/default/virtualbox/private_key vagrant@127.0.0.1 -p 2200
```

### reload settings

```
$ vagrant reload
```

# Ansible

### inventory file

```bash
$ cat hosts
my-server ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_ssh_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key⏎ 

$ ansible my-server -i hosts -m ping
my-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### ansible.cfg

```
[defaults]
inventory = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = False
```

```bash
$ cat hosts
my-server ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200⏎ 

$ ansible my-server -m ping
my-server | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

### debugging

```bash
$ ansible my-server -m ping -vvv
```

### command execution

```bash
$ ansible my-server -m command -a ps
my-server | CHANGED | rc=0 >>
  PID TTY          TIME CMD
16572 pts/0    00:00:00 sh
16573 pts/0    00:00:00 python3
16575 pts/0    00:00:00 ps
```

with space.

```bash
$ ansible my-server -a "ps -o pid"
my-server | CHANGED | rc=0 >>
  PID
16842
16843
16845
```

### execute Playbook

```bash
$ ansible-playbook web-notls.yml
PLAY [nginx server with no TLS settings] ************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************
ok: [my-server]

TASK [install nginx] ********************************************************************************************************************************************************************
changed: [my-server]

TASK [copy nginx config file] ***********************************************************************************************************************************************************
changed: [my-server]

TASK [enable configuration] *************************************************************************************************************************************************************
ok: [my-server]

TASK [copy index.html] ******************************************************************************************************************************************************************
changed: [my-server]

TASK [restart nginx] ********************************************************************************************************************************************************************
changed: [my-server]

PLAY RECAP ******************************************************************************************************************************************************************************
my-server                  : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
$ curl http://localhost:8080
<p>Hello Ansible</p>⏎ 
```

# Nginx

## TLS

### make TLS certificates

```bash
$ cd nginx
$ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout nginx.key -out nginx.crt
```

```bash
$ ansible-playbook web-tls.yml
```

### test connection
maybe error found because of self TLS certificates, but that's ok.

```bash
$ curl https://localhost:8443
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```