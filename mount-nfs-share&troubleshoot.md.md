# How to Mount the `/LOGS` NFS Share

## Step 1: Create the Mount Point

Create the local directory where the NFS share will be mounted.

```bash
mkdir /LOGS
```

---

# Step 2: Configure `/etc/fstab`

Edit the `/etc/fstab` file and add the following entry:

```text
10.220.2.83:/logs    /LOGS    nfs    defaults,_netdev    0 0
```

### Explanation

| Field               | Description                                                                         |
| ------------------- | ----------------------------------------------------------------------------------- |
| `10.220.2.83:/logs` | NFS server and exported directory                                                   |
| `/LOGS`             | Local mount point                                                                   |
| `nfs`               | Filesystem type                                                                     |
| `defaults,_netdev`  | Standard mount options. `_netdev` ensures the network is available before mounting. |
| `0 0`               | Disable dump and filesystem check                                                   |

---

# Step 3: Mount the Filesystem

Run:

```bash
mount -a
```

If successful, the NFS share will be mounted.

---

# Troubleshooting

## Problem

If the following error appears:

```text
[root@ngd-dc2-npsb-spg01 /]# mount -a
mount: /LOGS: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
```

This usually indicates that the NFS client utilities are not installed or the NFS mount helper is missing.

---

# Step 1: Verify `nfs-utils` is Installed

Check whether the package is installed:

```bash
rpm -q nfs-utils
```

If it is not installed, install it using:

```bash
dnf install nfs-utils
```

---

# Step 2: Start and Enable the NFS Client Service

Start the NFS client service:

```bash
systemctl start nfs-client.target
```

Enable it to start automatically on boot:

```bash
systemctl enable nfs-client.target
```

---

# Step 3: Retry the Mount

Run:

```bash
mount -a
```

---

# Step 4: Check for NFS Errors

If the mount still fails, review the system logs.

### Journal Logs

```bash
journalctl -xe | grep -i nfs
```

### Kernel Messages

```bash
dmesg | grep -i nfs
```

These commands may provide additional information about the cause of the failure.

---

# Step 5: Verify the NFS Mount Helper

The NFS mount helper should exist on the system.

Check using either of the following commands:

```bash
ls -l /sbin/mount.nfs
```

or

```bash
which mount.nfs
```

Expected output:

```text
/sbin/mount.nfs
```

If the helper does not exist, it indicates that the `nfs-utils` package is missing or was not installed correctly.

---

# Verification

Confirm that the NFS share has been mounted successfully.

```bash
mount | grep /LOGS
```

or

```bash
df -h | grep /LOGS
```

Expected output should show that `/LOGS` is mounted from:

```text
10.220.2.83:/logs
```

---

# Summary

1. Create the mount point.
2. Add the NFS entry to `/etc/fstab`.
3. Run `mount -a`.
4. If mounting fails:

   * Verify `nfs-utils` is installed.
   * Install `nfs-utils` if necessary.
   * Start and enable `nfs-client.target`.
   * Verify `/sbin/mount.nfs` exists.
   * Check `journalctl` and `dmesg` for NFS-related errors.
5. Verify the mount using `mount` or `df -h`.
