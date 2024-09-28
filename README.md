# SFTP-with-OpenSSH
Secure SFTP Server Setup with OpenSSH

This guide provides a step-by-step walkthrough for setting up a secure SFTP server using OpenSSH on a Linux system. It's designed to be accessible for users of all levels—**beginners**, **intermediate**, and **advanced**.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Update System Packages](#step-1-update-system-packages)
- [Step 2: Install OpenSSH Server](#step-2-install-openssh-server)
- [Step 3: Create SFTP Group and User](#step-3-create-sftp-group-and-user)
- [Step 4: Configure SSH for SFTP Access](#step-4-configure-ssh-for-sftp-access)
- [Step 5: Set Directory Permissions](#step-5-set-directory-permissions)
- [Step 6: Implement SSH Key Authentication](#step-6-implement-ssh-key-authentication)
- [Step 7: Restart SSH Service](#step-7-restart-ssh-service)
- [Step 8: Test the SFTP Connection](#step-8-test-the-sftp-connection)
- [Security Measures Implemented](#security-measures-implemented)
- [Additional Resources](#additional-resources)

---

## Prerequisites

- A Linux server with **sudo** privileges.
- Basic knowledge of Linux command-line operations.
- Access to the server via SSH.

---

## Step 1: Update System Packages

Before starting, ensure your system packages are up-to-date.

```
sudo apt update && sudo apt upgrade -y
```

## Step 2: Install OpenSSH Server

Install the OpenSSH server package to enable SSH and SFTP functionalities.

```
sudo apt install openssh-server -y
```

## Step 3: Create SFTP Group and User

Create a dedicated group and user for SFTP access.

### Create SFTP Group

```
sudo groupadd sftp_users
```

### Create SFTP User
```
sudo useradd -m -G sftp_users -s /usr/sbin/nologin sftpuser
sudo passwd sftpuser
```
  - `-m`: Creates a home directory.
  - `-G sftp_users`: Adds the user to the `sftp_users` group.
  - `-s /usr/sbin/nologin`: Disables shell access.

## Step 4: Configure SSH for SFTP Access

Modify the SSH daemon configuration to set up SFTP-specific settings.

```
sudo nano /etc/ssh/sshd_config
```

Add the following at the end of the file:
```
Match Group sftp_users
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    X11Forwarding no
    AllowTcpForwarding no
```
  - `ChrootDirectory`: Restricts users to their home directory.
  - `ForceCommand internal-sftp`: Forces the use of SFTP.
  - `X11Forwarding no` and `AllowTcpForwarding no`: Enhance security by disabling unnecessary features.

## Step 5: Set Directory Permissions

Adjust permissions to secure the user's home directory.

```
sudo chown root:root /home/sftpuser
sudo chmod 755 /home/sftpuser
sudo mkdir /home/sftpuser/upload
sudo chown sftpuser:sftp_users /home/sftpuser/upload
```
  - Set Ownership: Root owns the home directory to prevent modifications.
  - User Upload Directory: The upload directory is owned by the user for read/write access.

## Step 6: Implement SSH Key Authentication

Enhance security by using SSH keys instead of passwords.

### On the Client Machine

Generate an SSH key pair:

```
ssh-keygen -t rsa -b 4096
```
Copy the public key to the server:
```
ssh-copy-id sftpuser@server_ip
```
### On the Server

Ensure the `.ssh` directory and `authorized_keys` file are correctly set up in `/home/sftpuser/.ssh/`.

```
sudo mkdir /home/sftpuser/.ssh
sudo chown sftpuser:sftp_users /home/sftpuser/.ssh
sudo chmod 700 /home/sftpuser/.ssh
```
## Step 7: Restart SSH Service

Apply the changes by restarting the SSH service.

```
sudo systemctl restart sshd
```
## Step 8: Test the SFTP Connection

From the client machine, test the SFTP access.

```
sftp sftpuser@server_ip
```
You should be connected to the SFTP server without shell access and confined to the `upload` directory.

## Security Measures Implemented

  - **SSH Key Authentication**: Prevents unauthorized access via password brute-forcing.
  - **Disabled Shell Access**: Users cannot execute shell commands.
  - **Chroot Jail**: Users are confined to their home directories.
  - **Strict Permissions**: Limits the potential impact of compromised accounts.
  - **Disabled Unnecessary Features**: `X11Forwarding` and `AllowTcpForwarding` are turned off.

---

## Additional Resources

- **OpenSSH Manual**: [OpenSSH Documentation](https://www.openssh.com/manual.html)
- **SFTP Clients**:
  - [FileZilla](https://filezilla-project.org/)
  - [WinSCP](https://winscp.net/)
  - [Cyberduck](https://cyberduck.io/)

---

