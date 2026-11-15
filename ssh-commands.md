# SSH Commands – Role-Based Ansible Practical

## SSH Architecture

The Amazon Linux Control Node initiates SSH connections to:
- Amazon Linux Managed Node
- Ubuntu Managed Node

**Protocol:** SSH  
**TCP port:** 22  
**Authentication:** EC2 key pair / OpenSSH-compatible private key

## 1. PPK to OpenSSH-Compatible Key

The practical started with a PuTTY `.ppk` key. PuTTYgen was used on Windows to:
1. Load the existing `.ppk` private key.
2. Export it in an OpenSSH-compatible private-key format.
3. Copy the converted key to the Control Node.

This is a format conversion of the existing private key, not creation of a new key pair.

## 2. Copy the Key to the Control Node (SCP)

Run from Windows PowerShell, adjusting local key path and Control Node public IP:

```powershell
scp -i .\master-key.pem .\ubuntu-ansible ec2-user@<CONTROL_NODE_PUBLIC_IP>:/home/ec2-user/
```

The example is based on the command recorded in the practical. It assumes the local `master-key.pem` authenticates to the Control Node and `ubuntu-ansible` is the converted key being copied.

## 3. Restrict Key Permissions on the Control Node

```bash
chmod 400 /home/ec2-user/ubuntu-ansible
```

## 4. Connect to the Ubuntu Managed Node

```bash
ssh -i /home/ec2-user/ubuntu-ansible ubuntu@<UBUNTU_PUBLIC_IP>
```

## 5. Connect to the Amazon Linux Managed Node

```bash
ssh -i /home/ec2-user/ubuntu-ansible ec2-user@<AMAZON_LINUX_PUBLIC_IP>
```

## 6. SSH Troubleshooting Checklist

- Confirm the Control Node can reach the Managed Node on TCP 22.
- Confirm the Managed Node Security Group allows SSH from the Control Node.
- Use the correct default login username (`ubuntu` for the Ubuntu AMI used in the lab; `ec2-user` for Amazon Linux).
- Confirm the private-key path exists on the Control Node.
- Confirm key permissions are restrictive (`chmod 400`).
- Confirm the key corresponds to the EC2 key pair selected when the instance was launched.

## Security

Never upload `.pem`, `.ppk`, or any private-key file to GitHub. Do not paste private-key contents into chat, documentation, or terminal screenshots.
