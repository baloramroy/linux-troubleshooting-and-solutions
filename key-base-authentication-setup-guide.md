# SSH Key-Based Authentication Setup Guide (Linux)

**Document Version:** 1.0\
**Purpose:** Configure passwordless SSH authentication using public/private key pairs between Linux servers.

---

# 1. Overview

SSH key-based authentication allows a user on one server (client) to securely log in to another server (SSH server) without entering a password.

Authentication is performed using a **public/private key pair**:

* **Private Key** – Stored securely on the client machine. Never share it.
* **Public Key** – Copied to the target server and stored in the user's `authorized_keys` file.

---

# 2. Architecture

```
+------------------------+                 +-------------------------+
|      Client Server     |                 |      SSH Server         |
|------------------------|                 |-------------------------|
| User: user1            |                 | User: user2             |
|                        |                 |                         |
| ~/.ssh/id_rsa          | --------------> | ~/.ssh/authorized_keys  |
| ~/.ssh/id_rsa.pub      |  (Public Key)   |                         |
|                        |                 |                         |
| Private Key stays here |                 | Public Key stored here  |
+------------------------+                 +-------------------------+
```

Only the **public key** is copied to the server.

The **private key never leaves the client.**

---

# 3. Prerequisites

* SSH server installed
* SSH service running
* User account exists on the SSH server
* Network connectivity
* TCP port 22 (or custom SSH port) reachable

---

# 4. Generate SSH Key Pair (Client)

If no SSH key exists:

```bash
ssh-keygen -t ed25519 -C "user1@client"
```

or RSA:

```bash
ssh-keygen -t rsa -b 4096
```

Press Enter for defaults.

Generated files:

```
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

or

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

# 5. View the Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

or

```bash
cat ~/.ssh/id_rsa.pub
```

Example:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBY.... user1@client
```

Copy the entire line.

---

# 6. Prepare the Target User on SSH Server

Assume:

```
Client User : user1
Server User : user2
```

Login to the SSH server as root or with sudo.



## Create the .ssh Directory

```bash
mkdir -p /home/user2/.ssh
```



## Set Ownership

```bash
chown user2:user2 /home/user2/.ssh
```



## Set Directory Permission

```bash
chmod 700 /home/user2/.ssh
```



## Create authorized_keys

```bash
touch /home/user2/.ssh/authorized_keys
```



## Paste Public Key

Edit:

```bash
vi /home/user2/.ssh/authorized_keys
```

Paste:

```
ssh-ed25519 AAAAC3Nza..... user1@client
```

Save the file.



## Set File Permission

```bash
chmod 600 /home/user2/.ssh/authorized_keys
```



## Set Ownership

```bash
chown user2:user2 /home/user2/.ssh/authorized_keys
```

---

# 7. Verify SSH Server Configuration

Open:

```bash
vi /etc/ssh/sshd_config
```

Verify:

```
PubkeyAuthentication yes

AuthorizedKeysFile .ssh/authorized_keys

PasswordAuthentication yes
```

> **Note:** `PasswordAuthentication` may remain enabled during testing. Once key-based login is confirmed, you can disable it if desired.

---

# 8. Restart SSH (Only if Configuration Changed)

If you modified `sshd_config`, validate it:

```bash
sshd -t
```

No output means the configuration is valid.

Reload:

```bash
systemctl reload sshd
```

or

```bash
systemctl restart sshd
```

> **Note:** If you only created the `.ssh` directory or updated `authorized_keys`, **no reload or restart is required**. SSH reads `authorized_keys` for each new connection.

---

# 9. Connect to the Server

Default SSH port:

```bash
ssh user2@server_ip
```

Custom SSH port:

```bash
ssh -p 40167 user2@server_ip
```

Specific private key:

```bash
ssh -i ~/.ssh/id_ed25519 -p 40167 user2@server_ip
```

---

# 10. First Connection Warning

On the first connection, SSH may display:

```
The authenticity of host 'server' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

This is **normal**.

It means your client has never connected to this server before.

If the server is trusted, type:

```
yes
```

SSH stores the server's host key in:

```
~/.ssh/known_hosts
```

Future connections will not show this prompt unless the server's host key changes.

---

# 11. Verify Server Fingerprint (Optional)

On the SSH server:

```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

Compare the fingerprint with the one shown by the SSH client before accepting it.

---

# 12. Test Authentication

Run:

```bash
ssh -v user2@server_ip
```

Successful authentication includes lines similar to:

```
Offering public key
Server accepts key
Authentication succeeded (publickey)
```

---

# 13. Common Issues

| Issue                                 | Cause                       | Solution                                           |
| ------------------------------------- | --------------------------- | -------------------------------------------------- |
| Permission denied (publickey)         | Public key not installed    | Verify `authorized_keys`                           |
| Permission denied                     | Wrong permissions           | Set `700` on `.ssh` and `600` on `authorized_keys` |
| Authentication fails                  | Wrong owner                 | `chown -R user2:user2 /home/user2/.ssh`            |
| Login prompts for password            | Public key not accepted     | Use `ssh -v` to troubleshoot                       |
| `.ssh` missing                        | New user                    | Create it manually                                 |
| First connection authenticity warning | Server not in `known_hosts` | Type `yes` after verifying the fingerprint         |

---

# 14. Verify Permissions

```bash
ls -ld /home/user2/.ssh
ls -l /home/user2/.ssh
```

Expected output:

```
drwx------ user2 user2 .ssh

-rw------- user2 user2 authorized_keys
```

---

# 15. Security Best Practices

* Never share the private key.
* Protect private keys with a passphrase where appropriate.
* Use `ed25519` keys for new deployments.
* Restrict permissions on `.ssh` and `authorized_keys`.
* Disable password authentication after verifying key-based authentication (if organizational policy allows).
* Regularly remove unused public keys from `authorized_keys`.
* Verify host key fingerprints before accepting first-time connections.

---

# 16. Quick Reference

### On the Server

```bash
mkdir -p /home/user2/.ssh
touch /home/user2/.ssh/authorized_keys
chmod 700 /home/user2/.ssh
chmod 600 /home/user2/.ssh/authorized_keys
chown -R user2:user2 /home/user2/.ssh
```

### On the Client

```bash
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub
ssh -p 40167 user2@server_ip
```

### Test

```bash
ssh -v -p 40167 user2@server_ip
```

This procedure follows standard OpenSSH practices and is suitable for RHEL, CentOS, Rocky Linux, AlmaLinux, Ubuntu, Debian, and most other Linux distributions.
