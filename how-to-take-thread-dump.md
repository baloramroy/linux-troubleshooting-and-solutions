# How to Take a Java Thread Dump on RHEL/Linux

## What is a Thread Dump?

A **thread dump** is a snapshot of all the threads running inside a Java Virtual Machine (JVM) at a specific point in time.

It is commonly used to troubleshoot:

* High CPU usage
* Application hangs
* Deadlocks
* Slow performance
* Unresponsive applications

---

## Step 1: Verify the Java Installation

Determine which Java installation the application is using.

```bash
which java
```

Example:

```text
/bin/java
```

Resolve the actual Java path:

```bash
readlink -f $(which java)
```

Example:

```text
/opt/jdk1.8.0_161/bin/java
```

Check the Java version:

```bash
java -version
```

Example:

```text
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```

---

## Step 2: Locate `jstack`

Verify whether the `jstack` utility is available.

```bash
find / -type f -name jstack 2>/dev/null
```

Or check directly inside the JDK directory:

```bash
ls -l /opt/jdk1.8.0_161/bin/jstack
```

Example:

```text
-rwxr-xr-x 1 root root 123456 Jan 10 2024 /opt/jdk1.8.0_161/bin/jstack
```

If `jstack` is not found, the server may only have a JRE installed instead of a full JDK.

---

## Step 3: Find the Java Process ID (PID)

List the Java processes.

```bash
ps -ef | grep <process-name>
```

Example:

```text
70332 org.apache.catalina.startup.Bootstrap
```

The PID in this example is:

```text
70332
```

---

## Step 4: Take a Normal Thread Dump

Run:

```bash
/opt/jdk1.8.0_161/bin/jstack -l 70332 > /tmp/thread_dump.txt
```

or with a timestamp:

```bash
/opt/jdk1.8.0_161/bin/jstack -l 70332 > /tmp/thread_dump_$(date +%F_%H-%M-%S).txt
```

**Note:**

* Use **`-l`** (lowercase L), **not** **`-I`** (uppercase i).

---

## Step 5: If `jstack` Fails

If you receive errors such as:

```text
Unable to open socket file
```

or

```text
Unable to attach to process
```

try forcing the thread dump:

```bash
/opt/jdk1.8.0_161/bin/jstack -F 70332 > /tmp/thread_dump.txt
```

The `-F` option forces `jstack` to attach to the JVM. It is useful when the JVM is hung or not responding.

> **Note:** The `-F` option is supported only on some platforms and JVM implementations.

---

## Alternative Method: Using `kill -3`

If `jstack` is unavailable, send the `SIGQUIT` signal to the Java process.

```bash
kill -3 70332
```

This **does not terminate** the Java application.

The JVM writes the thread dump to its standard output, which is typically redirected to one of the following:

* `catalina.out`
* `server.log`
* `nohup.out`
* Application log
* Systemd journal

---

## Verify the Thread Dump

If using `jstack`, verify that the output file was created.

```bash
ls -lh /tmp/thread_dump*
```

View the beginning of the thread dump:

```bash
head -50 /tmp/thread_dump.txt
```

Or inspect it with:

```bash
less /tmp/thread_dump.txt
```

---

## Common Errors

### No such file or directory

Example:

```text
/opt/jdk1.8.0_491/bin/jstack: No such file or directory
```

**Cause:**

The specified JDK path is incorrect.

**Solution:**

Determine the active Java installation:

```bash
readlink -f $(which java)
```

Then use the `jstack` binary from that JDK.

---

### `jstack` Not Found

Check:

```bash
find / -type f -name jstack 2>/dev/null
```

If nothing is returned, the server likely has only a JRE installed. Use `kill -3` or install a JDK.

---

## Useful Commands Summary

```bash
# Find Java executable
which java

# Show actual Java installation
readlink -f $(which java)

# Check Java version
java -version

# Locate jstack
find / -type f -name jstack 2>/dev/null

# Verify jstack in JDK
ls -l /opt/jdk1.8.0_161/bin/jstack

# Find Java PID
jps -l
ps -ef | grep java

# Take normal thread dump
/opt/jdk1.8.0_161/bin/jstack -l <PID> > /tmp/thread_dump.txt

# Force thread dump
/opt/jdk1.8.0_161/bin/jstack -F <PID> > /tmp/thread_dump.txt

# Alternative method
kill -3 <PID>
```

## Best Practices

* Prefer `jstack -l` for routine troubleshooting.
* Use `jstack -F` only if the JVM is unresponsive.
* Take **3 to 5 thread dumps** at **10-second intervals** when investigating hangs or performance issues.
* Preserve the thread dump files before restarting the application.
* Ensure you have permission to attach to the target Java process (typically by running as the same user or as `root`).
