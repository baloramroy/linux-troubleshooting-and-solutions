## Keyless Authentication Setup

Yes. In your case the SSH direction is:

**10.10.10.10**

* User: `parser`
* This is the **source/client**

⬇️ SSH key authentication

**10.10.10.11**

* User: `backloguser`
* This is the **destination/server**

The important point is: **generate the SSH key as `parser` on 10.10.10.10**, and put **only the public key** into `backloguser`'s `~/.ssh/authorized_keys` on 10.10.10.11. This is the standard OpenSSH setup. ([Red Hat Documentation][1])

### 1. Login to 10.10.10.10 as `parser`

```bash
ssh parser@10.10.10.10
```

Verify:

```bash
whoami
hostname
```

You should see:

```text
parser
10.10.10.10
```

### 2. Check whether `parser` already has an SSH key

```bash
ls -la ~/.ssh/
```

Look for something like:

```text
id_ed25519
id_ed25519.pub
```

or:

```text
id_rsa
id_rsa.pub
```

If there is **no suitable key**, generate one.

I recommend Ed25519 on modern RHEL/OpenSSH systems:

```bash
ssh-keygen -t ed25519
```

When asked:

```text
Enter file in which to save the key (/home/parser/.ssh/id_ed25519):
```

Press **Enter**.

For the passphrase:

```text
Enter passphrase (empty for no passphrase):
```

For a server-to-server automation scenario, you would normally leave it empty if the key must work unattended, for example from a script/cron/job.

This creates:

```text
/home/parser/.ssh/id_ed25519       <-- PRIVATE KEY
/home/parser/.ssh/id_ed25519.pub   <-- PUBLIC KEY
```

**Never copy `id_ed25519` to 10.10.10.11.** Only the `.pub` file goes there. ([Red Hat Documentation][2])

### 3. Copy the public key to `backloguser`

From **10.10.10.10**, run:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub backloguser@10.10.10.11
```

It will ask for the current password of `backloguser` on `10.10.10.11`.

After successful authentication, `ssh-copy-id` adds the public key to:

```text
/home/backloguser/.ssh/authorized_keys
```

This is the easiest method recommended by Red Hat documentation. ([Red Hat Documentation][1])

### 4. Test it

Still on **10.10.10.10**, as `parser`:

```bash
ssh backloguser@10.10.10.11
```

If everything is correct, you should get into 10.10.10.11 **without being asked for `backloguser`'s password**.

You can also explicitly test public-key authentication:

```bash
ssh -o PreferredAuthentications=publickey backloguser@10.10.10.11
```

### 5. Check the destination if `ssh-copy-id` isn't available

On **10.10.10.10**:

```bash
cat ~/.ssh/id_ed25519.pub
```

You'll get one long line beginning with something like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...
```

Then on **10.10.10.11**, as `backloguser`:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

Add the public key:

```bash
vi ~/.ssh/authorized_keys
```

Paste the **entire single line** from `id_ed25519.pub`.

Then:

```bash
chmod 600 ~/.ssh/authorized_keys
```

Correct ownership is also important:

```bash
chown -R backloguser:backloguser ~/.ssh
```

Red Hat specifically notes that incorrect `.ssh`/`authorized_keys` permissions can cause key authentication to fall back to password authentication. ([Red Hat Developer][3])

### 6. Do NOT disable password authentication yet

You **do not need to modify `/etc/ssh/sshd_config`** just to establish key-based authentication.

First make sure this works:

```bash
ssh backloguser@10.10.10.11
```

Only after successful testing would you consider disabling password authentication globally:

```text
PasswordAuthentication no
```

Red Hat recommends testing key authentication before disabling password authentication, especially when working remotely. ([Red Hat Documentation][2])

### Your final setup

```text
10.10.10.10
┌─────────────────────────────┐
│ parser                      │
│                             │
│ ~/.ssh/                     │
│   ├── id_ed25519       🔒   │  PRIVATE
│   └── id_ed25519.pub        │  PUBLIC
└──────────────┬──────────────┘
               │
               │ SSH
               │
               │ public-key authentication
               ▼
10.10.10.11
┌─────────────────────────────┐
│ backloguser                 │
│                             │
│ ~/.ssh/                     │
│   └── authorized_keys       │
│       └── parser's PUBLIC   │
│           KEY                │
└─────────────────────────────┘
```

So the **short version** is:

```bash
# 10.10.10.10 as parser
ssh-keygen -t ed25519

# Copy public key to 10.10.10.11
ssh-copy-id -i ~/.ssh/id_ed25519.pub backloguser@10.10.10.11

# Test
ssh backloguser@10.10.10.11
```
