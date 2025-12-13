
#

## **/var** – Variable data

Contains data that **changes frequently**.

Inside:

### **/var/log**

System log files
• `/var/log/messages`
• `/var/log/secure`
• `/var/log/audit/`

### **/var/spool**

Spool files (mail, print jobs)

### **/var/lib**

Dynamic state information for services
• `/var/lib/mysql/`
• `/var/lib/docker/`
• `/var/lib/rpm/`

### **/var/cache**

Cached data for applications
• `/var/cache/dnf/`

---

## **/media** – Automated mount directory

Used by the system to auto-mount removable devices like USBs, DVDs.

Example:
• `/media/user/USB_DRIVE`

---

## **/dev** – Device files

Contains device nodes representing hardware.

Examples:
• `/dev/sda` – disk
• `/dev/tty` – terminal
• `/dev/null` – null device

Created dynamically by udev.

---

## **/proc** – Kernel information (virtual filesystem)

Not real files—these represent kernel runtime information.

Examples:
• `/proc/cpuinfo`
• `/proc/meminfo`
• `/proc/uptime`

---

## **/sys** – Kernel device tree (virtual filesystem)

Similar to /proc but more organized.

Used to manage kernel hardware info and device attributes.

---

## **/run** – Runtime data

Contains information that is only valid during boot session.

Example:
• PID files (`/run/*.pid`)
• Socket files

---

## **/srv** – Service data

Data used by system services.

Example:
• `/srv/www/` – web server data
• `/srv/ftp/` – FTP server data

Not widely used on all distros.

---

## **/tmp** – Temporary files

World-writable temporary files.

Rules:
• Can be deleted anytime
• Often cleaned on reboot
• Permissions usually `1777` (sticky bit)

---

# **5. FHS and SELinux**

SELinux follows FHS paths to define default contexts.

Examples:
• `/var/www/html` → httpd_sys_content_t
• `/var/log` → var_log_t

You should not place service data outside expected FHS directories unless you properly change SELinux labels.

---

# **6. FHS and Docker / Containerization**

Inside containers, a minimal FHS is used:

A container usually includes:

• `/bin`
• `/sbin`
• `/usr`
• `/etc`
• `/var`

But does **not** need:
• `/boot`
• `/dev` (virtualized)
• `/proc` (runtime mount)

---

# **7. Real-Life Usage for Sysadmins**

• Know where logs are stored → `/var/log`
• Know where configs are stored → `/etc`
• Know where binaries are → `/bin`, `/usr/bin`
• Know where service data is → `/var/lib/service`
• Know where to mount disks → `/mnt`, `/media`
• Know where to install custom software → `/usr/local`, `/opt`

This knowledge is essential for RHCSA / RHCE.

---
