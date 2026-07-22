# Packets Capture (pcap) procedure
To capture packets on your server, you'll typically use `tcpdump`.

Your interface is:

```text
ens33
```

---

## 1. Verify tcpdump is installed

```bash
which tcpdump
```

or

```bash
tcpdump --version
```

If it's not installed (RHEL/CentOS):

```bash
sudo dnf install tcpdump
```

or

```bash
sudo yum install tcpdump
```

---

# 2. Capture everything

```bash
sudo tcpdump -i ens33 -w /tmp/apigw_capture.pcap
```

Press:

```
Ctrl + C
```

to stop.

---

# 3. Capture only 1000 packets

```bash
sudo tcpdump -i ens33 -c 1000 -w /tmp/apigw_capture.pcap
```

---

# 4. Capture for 60 seconds

Using `timeout`:

```bash
sudo timeout 60 tcpdump -i ens33 -w /tmp/apigw_capture.pcap
```

---

# 5. Capture only one host

Suppose the backend server is

```
10.210.110.50
```

```bash
sudo tcpdump -i ens33 host 10.210.110.50 -w /tmp/backend.pcap
```

---

# 6. Capture one port

Example:

```bash
sudo tcpdump -i ens33 port 8080 -w /tmp/8080.pcap
```

or

```bash
sudo tcpdump -i ens33 port 443 -w /tmp/https.pcap
```

---

# 7. Capture multiple ports

Example:

```bash
sudo tcpdump -i ens33 'port 80 or port 443'
```

or save it:

```bash
sudo tcpdump -i ens33 'port 80 or port 443' -w /tmp/web.pcap
```

---

# 8. Capture traffic from one source

```bash
sudo tcpdump -i ens33 src host 10.210.110.25
```

---

# 9. Capture traffic to one destination

```bash
sudo tcpdump -i ens33 dst host 10.210.110.25
```

---

# 10. Capture both directions

```bash
sudo tcpdump -i ens33 host 10.210.110.25
```

---

# 11. Show packets on screen (don't save)

```bash
sudo tcpdump -i ens33
```

For more detail:

```bash
sudo tcpdump -nnvvv -i ens33
```

Where:

* `-n` = Don't resolve hostnames.
* `-nn` = Don't resolve hostnames or service names.
* `-vvv` = Maximum verbosity.

---

# 12. Capture only TCP

```bash
sudo tcpdump -i ens33 tcp -w /tmp/tcp.pcap
```

---

# 13. Capture only UDP

```bash
sudo tcpdump -i ens33 udp -w /tmp/udp.pcap
```

---

# 14. Capture with a file size limit (rotating files)

Each file 100 MB:

```bash
sudo tcpdump -i ens33 -C 100 -W 5 -w /tmp/apigw.pcap
```

This creates:

```
apigw.pcap0
apigw.pcap1
apigw.pcap2
apigw.pcap3
apigw.pcap4
```

Maximum disk usage: about 500 MB.

---

# 15. Capture full packets (recommended)

```bash
sudo tcpdump -i ens33 -s 0 -w /tmp/apigw_capture.pcap
```

The `-s 0` option captures the entire packet instead of truncating it.

---

# 16. Capture with timestamps

```bash
sudo tcpdump -tttt -i ens33
```

---

## My recommendation for troubleshooting your API Gateway

For production troubleshooting, I usually recommend:

```bash
sudo tcpdump -i ens33 -s 0 -nn -w /tmp/apigw_$(date +%F_%H%M%S).pcap
```

Or, if you want to limit the capture to 5 minutes:

```bash
sudo timeout 300 tcpdump -i ens33 -s 0 -nn -w /tmp/apigw_$(date +%F_%H%M%S).pcap
```

This:

* Captures on `ens33`
* Saves full packets (`-s 0`)
* Avoids DNS/service-name resolution (`-nn`)
* Produces a timestamped filename
* Stops automatically after 5 minutes (with `timeout`)

If your goal is to troubleshoot a specific issue with the API Gateway (`kpp-apigw-INST_1`/`INST_2`), let me know:

* the client IP,
* the backend server IP,
* or the application port,

and I can help you build a much more targeted `tcpdump` filter so the capture contains only the relevant traffic.
