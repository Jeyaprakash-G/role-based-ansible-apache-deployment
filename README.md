# Role-Based Ansible Playbook – Apache Deployment

## Project Overview

This practical demonstrates how to use Ansible to manage two Linux servers from one Control Node and deploy the Apache web server using a reusable Ansible Role.

The Control Node runs Ansible. The Managed Nodes are accessed over SSH. A single playbook invokes the `apache` role, and OS-family conditions select the correct package manager and Apache package.

## Objectives

- Create and configure three AWS EC2 instances.
- Configure SSH key-based access from the Control Node to both Managed Nodes.
- Install Ansible on the Control Node.
- Define Managed Nodes in an Ansible inventory.
- Configure Ansible defaults using `ansible.cfg`.
- Test SSH and Ansible connectivity.
- Create an `apache` role using `ansible-galaxy`.
- Install Apache on Amazon Linux and Ubuntu using OS-specific conditions.
- Start the Apache service on both Managed Nodes.
- Verify the deployed Apache default page in a browser.

## Lab Architecture

| Instance | Operating system | Role |
|---|---|---|
| Instance 1 | Amazon Linux | Control Node / Master |
| Instance 2 | Amazon Linux | Managed Node |
| Instance 3 | Ubuntu Linux | Managed Node |

**Connectivity:** Control Node → Managed Nodes over SSH (TCP port 22).

See [`architecture/role-based-ansible-architecture.pdf`](architecture/role-based-ansible-architecture.pdf).

## Technology Used

- AWS EC2 and Security Groups
- Amazon Linux and Ubuntu
- Ansible
- SSH / OpenSSH
- PuTTYgen (PPK to OpenSSH-compatible private key conversion)
- YAML
- Ansible Roles and Playbooks
- `yum` / `apt`
- Apache HTTP Server (`httpd` on Amazon Linux, `apache2` on Ubuntu)

## Repository Structure

```text
role-based-ansible-playbook/
├── README.md
├── .gitignore
├── ansible.cfg
├── hosts
├── roles.yml
├── roles/
│   └── apache/
│       ├── defaults/
│       ├── files/
│       ├── handlers/
│       ├── meta/
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       ├── tests/
│       └── vars/
└── docs/
    ├── commands-reference.md
    ├── ssh-commands.md
    └── troubleshooting.md
```

> The document names the playbook `roles.yml` and the inventory `hosts`. Keep the actual filenames used in your repository consistent with your files. If your local role folder is named `apache` at the project root rather than `roles/apache`, adjust `roles_path` and the structure accordingly.

## Execution Workflow

1. Launch the three EC2 instances.
2. Allow SSH (TCP 22) from the Control Node to both Managed Nodes.
3. Create the EC2 key pair. The lab key was in PuTTY `.ppk` format.
4. Use PuTTYgen to export an OpenSSH-compatible private key.
5. Copy the converted key to the Control Node using `scp`.
6. Restrict private-key permissions using `chmod 400`.
7. Install Ansible on the Control Node.
8. Create the `hosts` inventory with the Managed Nodes, usernames, and key path.
9. Configure `ansible.cfg`.
10. Test direct SSH and Ansible ping.
11. Create the Apache role.
12. Define OS-specific install tasks and a common service-start task.
13. Execute the playbook.
14. Open the Ubuntu Managed Node's public IP in a browser to verify the Apache2 default page. Also verify Apache on the Amazon Linux node.

## Important Configuration

### `ansible.cfg`

The documented example is:

```ini
[defaults]
inventory = ./hosts
remote_user = ec2-user
roles_path = ./roles
```

The inventory example specifies `ubuntu` for the Ubuntu host and `ec2-user` for the Amazon Linux host. Because these usernames differ, avoid applying a single global `remote_user` to both hosts; define `ansible_user` per host in the inventory or remove the global setting.

### `hosts` inventory

The document's example:

```ini
[ubuntu]
ubuntu1 ansible_host=<UBUNTU_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ec2-user/ubuntu-ansible

[linux]
linux1 ansible_host=<AMAZON_LINUX_PUBLIC_IP> ansible_user=ec2-user ansible_ssh_private_key_file=/home/ec2-user/ubuntu-ansible
```

Replace placeholders with your own instance addresses. Do not publish real public IPs, private IPs, or key paths if they reveal sensitive lab infrastructure.

### Apache role – `roles/apache/tasks/main.yml`

```yaml
---
- name: Install the latest version of Apache on Amazon Linux
  yum:
    name: httpd
    state: latest
  when: ansible_os_family == "RedHat"

- name: Install the latest version of Apache on Ubuntu
  apt:
    name: apache2
    state: latest
  when: ansible_os_family == "Debian"

- name: Start Apache service
  service:
    name: "{{ 'apache2' if ansible_os_family == 'Debian' else 'httpd' }}"
    state: started
```

### Playbook – `roles.yml`

```yaml
---
- name: Install and Start Apache Web Server
  hosts: all
  remote_user: ec2-user
  become: yes
  roles:
    - apache
```

**Inventory username note:** The source document uses `remote_user: ec2-user` in the playbook while its Ubuntu inventory example uses `ansible_user=ubuntu`. The per-host inventory username should take precedence; the cleanest configuration is to remove the common `remote_user` from the play and set each host's `ansible_user` in inventory.

## Run the Playbook

From the project directory:

```bash
ansible-playbook -i hosts roles.yml
```

Optional validation commands:

```bash
ansible-inventory -i hosts --list
ansible all -i hosts -m ping
ansible-playbook -i hosts roles.yml --syntax-check
ansible-playbook -i hosts roles.yml --check
```

## Verification

### Check services

Amazon Linux:

```bash
sudo systemctl status httpd
```

Ubuntu:

```bash
sudo systemctl status apache2
```

### Verify in a browser

Open the Ubuntu Managed Node's public IPv4 address in a browser using HTTP. The practical screenshots show the **Apache2 Ubuntu Default Page** (“It works!”), confirming that Apache was installed and serving its default page.

## What Was Practiced

- Control Node vs Managed Node
- SSH key-based authentication
- Inventory groups and host variables
- `ansible.cfg`
- Ansible ping module
- Role creation with `ansible-galaxy init`
- Role directory conventions
- OS facts: `ansible_os_family`
- Conditional execution using `when`
- Package installation using `yum` and `apt`
- Service management using the `service` module
- Playbook execution
- Basic deployment verification

## Next Learning Step

The practical document identifies Variables and Jinja2 Templates as the next step. It includes a proposed example using `vars/main.yml`, `templates/index.html.j2`, and the `template` module. These are presented as a next-step enhancement, not as completed work in this practical.

## Documentation

- [Commands Reference](docs/commands-reference.md)
- [SSH Commands](docs/ssh-commands.md)
- [Troubleshooting Log](docs/troubleshooting.md)
- [Architecture PDF](architecture/role-based-ansible-architecture.pdf)

## Security Notes Before Publishing

- Never commit `.pem`, `.ppk`, private keys, passwords, or AWS credentials.
- Replace real IP addresses with placeholders in public examples.
- Do not use `0.0.0.0/0` for SSH in a production security group. Restrict TCP 22 to the Control Node or a trusted administration CIDR.
- Keep private-key permissions restrictive (`chmod 400`).
- Review screenshots for account IDs, public IPs, private IPs, and other sensitive information before committing.

## Lab Scope and Evidence

This README is based on the supplied 40-page practical document and the previously discussed hands-on context. The document confirms the three-node lab, SSH setup, Ansible installation, inventory/configuration, Apache role, OS-based tasks, playbook execution, and Ubuntu Apache browser verification. It does not provide a complete recorded terminal transcript for every command or every error; the troubleshooting guide labels inferred checks separately from explicitly visible/documented issues.
