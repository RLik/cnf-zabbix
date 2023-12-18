# Example of configuring Zabbix server from box by GitLab pipeline

I use this Ansile playbookas a base to configuring Zabbix server out of box for my labs using GitLab pipeline. Playbook was tested on followins OS: Ubuntu Server 22.04.

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

- [ ] In GitLab Project settings add variale named $ANSIBLE_VAULT_PASSWORD and in value part specify password that you've use to encrype zabbix.yml file by Ansible Vault.
- [ ] Install additional packages for work with postgresql_db Ansible module on host with Ansible and gitlab runner. In my case this is GitLab server host.
```
sudo apt install python3-pip python-setuptools virtualenv python3-psycopg2
```
- [ ] (Optional if ssl required) Create directory for storing SSL cert and keys files. Add contents of SSL crt and ke files (or just copy this files to server):
```
sudo mkdir -p /etc/ansible/config/ssl
sudo nano /etc/ansible/config/ssl/itproblog.ru.key
sudo nano /etc/ansible/config/ssl/itproblog.ru.crt
```

- [ ] (Optional if ssl required) Give rights to read SSL cert and keys files only to root and gitlab-runner users:
```
sudo chmod -R 650 /etc/ansible/config/
sudo chown root:gitlab-runner -R /etc/ansible/config/
```

# Zabbix related configuration

## Prepare configuration
1. Change repo_download_url variable in file roles/zabbix/vars/main.yml to actual repo download URL.
2. Create variable $PG_ZABBIX_USER_PWD in GitLab Project settings.
3. Correct postgresql user name for zabbix (variable zabbix_db_user_name) in roles/zabbix/vars/main.yml if needed.
3. Correct postgresql databse name for zabbix (variable zabbix_db_name) in roles/zabbix/vars/main.yml if needed.
4. Correct server_name variable in roles/zabbix/vars/main.yml. Specify name by which Zabbix server will be available.
5. If you need HTTPS set enable_ssl to "true" in roles/zabbix/vars/main.yml and specify SSL cert and key file location on ansible and Zabbix server in vars: ssl_certificate_key_path_source, ssl_certificate_path_source, ssl_certificate_key_path and ssl_certificate_path

## Final step
Go to configured hostname (variable server_name n roles/zabbix/vars/main.yml) and complete setup.