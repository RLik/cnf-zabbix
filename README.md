# Example of configuring Zabbix server from box

I use this Ansile playbookas a base to configuring Zabbix server out of box for my labs. Playbook was tested on followins OS: Ubuntu Server 22.04.

Configuration: Zabbix Server, Zabbix Frontend (nginx) and Zabbix database server (postgresql).

You can freely use this solution, but without any guarantee from the author.

## Environment description

I use GitLab CE server with local shell agent and Ansible installed. GitLab runner run under gitlab-runner user account.

## Step to prepare

- [ ] Switch to gitlab-runner user context:
```
 sudo su - gitlab-runner
```
- [ ] Generate SSH keys if they wasn't generated early:
```
ssh-keygen
```
- [ ] Correct inventory file (inventory/inv.ini) and add appropriate hosts.
- [ ] Copy SSH public key to all servers form inventory file:
```
ssh-copy-id roman@10.10.10.44
```
- [ ] Add sudo password for all hosts.
```
sudo nano /etc/ansible/vault/zabbix.yml
```
Format is:
```
ansible_sudo_pass: P@$$word
```
- [ ] Encrypt file with sudo password by Ansible Vault:
```
sudo ansible-vault encrypt /etc/ansible/vault/zabbix.yml
```
- [ ] Give rights to read sudo password file only to root and gitlab-runner users:
```
sudo chown root:gitlab-runner /etc/ansible/vault/zabbix.yml
sudo chmod 640 /etc/ansible/vault/zabbix.yml
sudo chmod 650 /etc/ansible/vault/
```

- [ ] In GitLab Project settings add variale named $ANSIBLE_VAULT_PASSWORD and in value part specify password that you've use to encrype nginx.yml file by Ansible Vault.
- [ ] Create directory for storing SSL cert and keys files. Add contents of SSL crt and ke files (or just copy this files to server):
```
sudo mkdir -p /etc/ansible/config/ssl
sudo nano /etc/ansible/config/ssl/itproblog.ru.key
sudo nano /etc/ansible/config/ssl/itproblog.ru.crt
```

- [ ] Give rights to read SSL cert and keys files only to root and gitlab-runner users:
```
sudo chmod -R 650 /etc/ansible/config/
sudo chown root:gitlab-runner -R /etc/ansible/config/
```

# Zabbix related configuration

1. Change repo_download_url variable in file roles/zabbix/vars/main.yml to actual repo download URL.