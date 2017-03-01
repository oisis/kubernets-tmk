### Ansible playbooks for Kubernetes cluster management

#### Requirements
- Linux machine
- Ansible > 2.2
- VirtualBox - if you want to run it for test
- Vagrant - if you want to run it for test

#### Check out repo
```git clone https://github.com/oisis/kubernets-tmk && cd ./kubernets-tmk```

#### Run Kubernetes cluster on your local machine with Vagrant and VirtualBox

```vagrant up etcd1 etcd2 etcd3 k8s1 k8s2 node1 node2```

#### Setup your prod environment

- Copy configuration

```cp -r environments/vagrant environments/prod```

- Setup your environment

  Edit files inside directory environments/prod and setup proper values. The most important file is:
environments/prod/inventory - put proper IP addresses of your servers there.

- Generate your ssh keys

```ssh-keygen -t rsa -b 4096 -f ./.ansible/id_rsa```

- Deploy public keys on servers

```ssh-copy-id -i ./.ansible/id_rsa.pub <USERNAME>@<HOSTNAME>```

- Check server connection with Ansible, example:

```ansible -i environments/vagrant/inventory etcd -u vagrant --key-file=./.ansible/id_rsa -m ping```

- Run any command in ad-hoc mode - example: `whoami` on etcd hosts group

```ansible -i environments/vagrant/inventory etcd -u vagrant --key-file=./.ansible/id_rsa -a "whoami"```

#### Ansible-vault - secret variables management

```ansible-vault <FLAG> --vault-password-file=./.ansible/ansible_password <PATH_TO_FILE>```

Available FLAGS:
- create
- decrypt
- edit
- encrypt
- rekey
- view
