# Commands Reference – Role-Based Ansible Apache Practical

Commands below are collected from the practical document or are standard verification commands directly associated with the documented steps. Replace placeholders before use.

## 1. Control Node – Ansible Setup

```bash
pip3 install ansible
ansible --version
```

## 2. SSH Private-Key Permissions

After copying the converted OpenSSH-compatible key to the Control Node:

```bash
chmod 400 /home/ec2-user/ubuntu-ansible
ls -l /home/ec2-user/ubuntu-ansible
```

## 3. Inventory and Configuration Checks

```bash
pwd
ls -la
cat ansible.cfg
cat hosts
ansible-inventory -i hosts --list
```

Example `ansible.cfg` from the practical:

```ini
[defaults]
inventory = ./hosts
remote_user = ec2-user
roles_path = ./roles
```

## 4. SSH Connectivity

```bash
ssh -i /home/ec2-user/ubuntu-ansible ubuntu@<UBUNTU_PUBLIC_IP>
ssh -i /home/ec2-user/ubuntu-ansible ec2-user@<AMAZON_LINUX_PUBLIC_IP>
```

## 5. Ansible Connectivity

```bash
ansible all -i hosts -m ping
ansible ubuntu -i hosts -m ping
ansible linux -i hosts -m ping
```

Ansible's `ping` module is not an ICMP ping. It verifies that Ansible can connect and execute a module on the target.

## 6. Create and Inspect the Role

```bash
ansible-galaxy init apache
tree apache
```

If `tree` is not installed, use:

```bash
find apache -maxdepth 3 -type f -o -type d
```

## 7. Inspect and Validate YAML / Playbook

```bash
cat roles/apache/tasks/main.yml
cat roles.yml
ansible-playbook -i hosts roles.yml --syntax-check
ansible-playbook -i hosts roles.yml --list-hosts
ansible-playbook -i hosts roles.yml --list-tasks
```

## 8. Execute the Playbook

```bash
ansible-playbook -i hosts roles.yml
```

Optional dry run:

```bash
ansible-playbook -i hosts roles.yml --check
```

## 9. Verify Apache Services

Amazon Linux:

```bash
sudo systemctl status httpd
sudo systemctl is-active httpd
```

Ubuntu:

```bash
sudo systemctl status apache2
sudo systemctl is-active apache2
```

## 10. Useful Linux File and Directory Commands

```bash
pwd
ls -la
cd <directory>
mkdir -p roles/apache/tasks
vi hosts
vi ansible.cfg
vi roles.yml
vi roles/apache/tasks/main.yml
cat <file>
```

## Notes

- The document uses `roles.yml` as the playbook filename and `hosts` as the inventory filename.
- The document's sample inventory uses `ubuntu` for Ubuntu and `ec2-user` for Amazon Linux.
- The source screenshots show Ansible installed through Python/pip and the `apache` role being initialized.
- Do not copy real IP addresses or private key files into a public GitHub repository.
