# Installing VirtualBox Guest Additions

Those of us who use **VirtualBox** to prepare VMs all face a **common problem**: after creating the VM, we cannot **copy and paste** from the host to the VM. The VM window also cannot be made **full-screen**. Because of this, we cannot run Linux machines in a **full-screen window**.

To solve this problem, we need to install a package called **VirtualBox Guest Additions**. But the bigger issue is that this package also **refuses** to **install** by default. Because of this, lightweight software like VirtualBox gets **ignored** by people like me.

But today, I will tell you why VirtualBox Guest Additions **doesn't install** and how you can **solve this problem**.

---

## Why does it not install?

When you install VirtualBox Guest Additions inside a CentOS VM, VirtualBox needs to compile **kernel modules**. Such modules are:
- `vboxguest`
- `vboxsf`
- `vboxvideo`

> **Note:**  
*A kernel module (driver) is a small piece of code that plugs directly into the Linux kernel.*

VirtualBox cannot compile these modules unless the **required tools** are installed.

**Required Package Details:**
- `gcc` - GNU Compiler Collection (C compiler)
- `kernel-devel` - Kernel development files and headers
- `kernel-headers` - Header files for the kernel
- `dkms` - Dynamic Kernel Module Support
- `make` - Build automation tool

> *Now if you want to see what each part does during the time of installation you can explore the **bottom** section of this document or if you don't then skip that part and start straight from the installation part.*

---
## How to Solve the Problem?

**Enable the EPEL Repository**
- Some required packages are not available in the default Red Hat/CentOS repositories, so first enable the EPEL repo.
  ```bash
  dnf install epel-release
  ```

**Install All Necessary Build Tools**
- Now install the required packages for building the VirtualBox kernel modules:
  ```bash
  dnf install gcc kernel-devel kernel-headers dkms make
  ```

##

**Run Diagnostics (Check Kernel Version Matching)**

Now after install the required packages run a **test** to check that the install packages are **matched** with the **kernel** version.

- Check kernel version
  ```bash
  uname -r
  ```
  **Output:** `5.14.0-575.el9.x86_64`

- Then check installed kernel-devel version
  ```bash
  rpm -q kernel-devel
  ```
  **Output:** `5.14.0-565.el9.x86_64`

- If the versions DO NOT match, install the correct **kernel-devel** package:
  ```bash
  dnf install kernel-devel-$(uname -r)
  ```
  **Purpose:** Installs the exact kernel-devel version that matches your currently running kernel.

##
**Insert/mount the VirtualBox Guest Additions ISO**

1.From the VirtualBox app menu of the host (top menu for the VM window):  

- Insert the **Guest addition** cd image
  
  ```text
  Devices → Insert Guest Additions CD image…
  ```
  
2.Then in the guest (CentOS VM):

- Create mount point and mount CDROM
  ```bash
  mount /dev/cdrom /mnt/
  ```
  **Output:** This will mount the inserted cd image to the mnt directory

- If `/dev/cdrom` isn't available, try `/dev/sr0`:
  ```bash
  mount /dev/sr0 /mnt/
  ```

- List ISO contents to confirm
  ```bash
  ls -l /mnt/cdrom
  ```

##

**Install VirtualBox Guest Additions**

- First go to the `/mnt` directory
  ```bash
  cd /mnt/
  ```

- Now run this command to install **Guest Additions**
  ```bash
  sudo sh ./VBoxLinuxAdditions.run
  ```
##

**Reboot the VM and check service**
- Now restart the system
  ```bash
  shutdown -r now
  ```
  
- After reboot, check modules and services:
  ```bash
  lsmod | grep vbox
  ```
  
- or check the service
  ```bash
  systemctl status vboxadd-service
  ```
  
  > **Note:**  
  You should see the service is loaded and vboxservice running.

---

## Lets see what each part does during the time of installation

**1. VirtualBox tries to build a kernel module**

When you run:
```bash
sudo ./VBoxLinuxAdditions.run
```
**Output:** VirtualBox will try to compile like vboxsf (shared folders module).
##

**2. VirtualBox needs a compiler**

It uses gcc to compile the C source code. Without gcc, you would get:
```
gcc: command not found
```
##

**3. It needs kernel source**

To build a module, the build system must know:
- How the kernel is structured
- What functions it provides
- How its internal APIs look
- What compiler settings the kernel was built with

That's why VirtualBox looks for below folder to find all those mentioned details
```
/usr/src/kernels/5.14.0-284.el9.x86_64/
```
And this folder comes from kernel-devel. Without kernel-devel, error looks like:
```
Your kernel headers for kernel 5.14.0-284 are missing.
```
##

**4. It needs just kernel header files**

Some parts of the build process need only kernel interface definitions → `/usr/include/linux/*`. These come from kernel-headers.  

- Without kernel-headers:
  ```
  fatal error: linux/version.h: No such file or directory
  ```
##

**5. DKMS then registers the module**

VirtualBox registers the module with DKMS:  
**Reason:**  
- Later when you update your kernel, DKMS will auto-rebuild VirtualBox modules.  
- Without DKMS your VM breaks after kernel update.
##

**6. make runs the actual build**

make reads a Makefile and compiles everything.
- Example command VirtualBox runs internally:
  ```bash
  make KERN_DIR=/usr/src/kernels/5.14.0-284.el9.x86_64
  ```
So make runs the entire build steps.

---
