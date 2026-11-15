# Troubleshooting Log – Role-Based Ansible Apache Practical

This log separates issues/warnings visible in the supplied practical material from additional checks recommended for diagnosing the same setup. The document does not provide a fully readable transcript for every terminal issue, so no unverified error message is presented as a confirmed event.

## 1. Ansible Cannot Find the Intended Inventory

**Observed/documented symptom:** Ansible emitted a warning that it was looking for an inventory under `/home/ec2-user/.ansible/hosts`, followed by no-inventory / no-hosts-matched behavior in the earlier practice context.

**Likely cause:** The active `ansible.cfg` was not being read from the project directory, or its `inventory` path pointed to the wrong location.

**Checks:**

```bash
pwd
ls -l ansible.cfg hosts
ansible-config dump --only-changed
ansible-inventory -i hosts --list
```

**Resolution:**
- Run Ansible from the project directory containing `ansible.cfg`, or explicitly specify `-i hosts`.
- Ensure the config contains the correct relative path:

```ini
[defaults]
inventory = ./hosts
roles_path = ./roles
```

- Execute:

```bash
ansible-playbook -i hosts roles.yml
```

## 2. `ansible.cfg` Log File Permission Warning

**Observed/documented symptom:** The earlier practice context recorded a warning that `/var/log/ansible.log` was not writable.

**Cause:** The user running Ansible did not have permission to write to the configured log file.

**Checks:**

```bash
grep -n "log_path" ansible.cfg
ls -l /var/log/ansible.log
```

**Resolution options:**
- Remove/comment out `log_path` if file logging is not required for the lab; or
- Configure a log file in a directory writable by the current user.

Do not solve this by broadly weakening system log-directory permissions.

## 3. Incorrect SSH Username

**Configuration detail:** The inventory example uses `ansible_user=ubuntu` for Ubuntu and `ansible_user=ec2-user` for Amazon Linux. The playbook example also sets `remote_user: ec2-user` globally.

**Risk:** A common remote user may be incorrect for one of the OS targets.

**Resolution:** Define the correct username per host in `hosts` and remove the global `remote_user` from the playbook, or ensure the inventory host variables override it as intended.

## 4. SSH Authentication or Permission Failure

**Potential symptom:** SSH reports permission denied, cannot read key, or refuses a key with overly open permissions.

**Checks:**

```bash
ls -l /home/ec2-user/ubuntu-ansible
chmod 400 /home/ec2-user/ubuntu-ansible
ssh -i /home/ec2-user/ubuntu-ansible ubuntu@<UBUNTU_PUBLIC_IP>
ssh -i /home/ec2-user/ubuntu-ansible ec2-user@<AMAZON_LINUX_PUBLIC_IP>
```

Also verify TCP 22 is allowed from the Control Node to the Managed Node and that the correct login username is used.

## 5. Python Interpreter Discovery Warning

**Observed/documented context:** The screenshots show Ansible/Python environment output and interpreter-related warnings. The document does not preserve a complete exact warning string.

**Checks:**

```bash
ansible all -i hosts -m setup -a 'filter=ansible_python*'
python3 --version
```

**Resolution:** Confirm Python 3 is installed on each Linux Managed Node. If necessary, set `ansible_python_interpreter` to the actual Python 3 path for that host. Do not copy a guessed interpreter path; discover it on the target first.

## 6. Role Not Found

**Potential symptom:** Ansible reports that the `apache` role cannot be found.

**Checks:**

```bash
pwd
find . -maxdepth 3 -type d
grep -n "roles_path" ansible.cfg
```

**Resolution:**
- If the role is at `roles/apache`, configure `roles_path = ./roles`.
- If the role is at the project root (`./apache`), either move it under `roles/` or configure the role path to match the actual location.
- Keep the playbook role name as `apache`.

## 7. Apache Package Task Skipped

**Potential symptom:** The expected package installation task is skipped.

**Cause to check:** `when` condition does not match the managed node's OS family.

**Checks:**

```bash
ansible all -i hosts -m setup -a 'filter=ansible_os_family'
```

The practical uses:
- `RedHat` → `yum` package `httpd`
- `Debian` → `apt` package `apache2`

## 8. Apache Service Not Running

**Checks:**

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

If the service is not running, inspect the service status and system logs on the affected Managed Node before rerunning the playbook.

## 9. Browser Cannot Open the Apache Page

**Checks:**
- Confirm Apache is installed and service is active.
- Confirm the Managed Node Security Group allows inbound HTTP TCP 80 from the test client.
- Use the correct public IPv4 address.
- Confirm the browser URL uses `http://` for this lab's default HTTP setup.

The supplied screenshots show the Ubuntu Apache2 default page, which is evidence that the Ubuntu web server responded successfully.

## 10. Idempotency / Repeated Execution

The practical uses package state `latest` and service state `started`. Re-running the playbook should leave the package at the latest available version and the service running. A changed count of zero is expected only when no package update or configuration change is required; `state: latest` can legitimately change a host when a newer package becomes available.

## Evidence / Scope Notes

- Confirmed by the supplied document: three EC2 nodes, SSH TCP 22, PPK-to-OpenSSH conversion, key copy and `chmod 400`, Ansible installation, inventory/configuration, SSH and Ansible ping tests, Apache role, OS-family conditions, playbook run, and Ubuntu Apache default page.
- Earlier conversation context additionally recorded inventory-path and `/var/log/ansible.log` permission warnings.
- Items explicitly labelled “Potential” or “Checks” are diagnostic guidance, not claims that the exact error occurred during the practical.
