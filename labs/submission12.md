## Task 1

**shim `containerd-shim-kata-v2 --version`**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:/tmp# containerd-shim-kata-v2 --version
Kata Containers containerd shim (Golang): id: "io.containerd.kata.v2", version: 3.3.0, commit: 19eb45a27d5b666f8f012c13ba6b504cee4b97b1

```

**successful test run with `sudo nerdctl run --runtime io.containerd.kata.v2 ...`**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:/tmp# sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 uname -a
WARN[0000] cannot set cgroup manager to "systemd" for runtime "io.containerd.kata.v2" 
Linux 9e38e077ceec 6.1.62 #1 SMP Tue Mar  5 10:00:02 UTC 2024 x86_64 Linux
```

## Task 2

**Show juice-runc health check (HTTP 200 from port 3012)**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# curl -s -o /dev/null -w "juice-runc: HTTP %{http_code}\n" http://localhost:3012 | tee labs/lab12/runc/health.txt
juice-runc: HTTP 200
```

**Show Kata containers running successfully with `--runtime io.containerd.kata.v2`**
```
=== Kata Container Tests ===
Linux a243816d234b 6.1.62 #1 SMP Tue Mar  5 10:00:02 UTC 2024 x86_64 Linux
6.1.62
model name	: Intel Xeon Processor (Skylake, IBRS)
```

**Compare kernel versions:**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== Kernel Version Comparison ===" | tee labs/lab12/analysis/kernel-comparison.txt
echo -n "Host kernel (runc uses this): " | tee -a labs/lab12/analysis/kernel-comparison.txt
uname -r | tee -a labs/lab12/analysis/kernel-comparison.txt
echo -n "Kata guest kernel: " | tee -a labs/lab12/analysis/kernel-comparison.txt
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 cat /proc/version | tee -a labs/lab12/analysis/kernel-comparison.txt

=== Kernel Version Comparison ===
Host kernel (runc uses this): 6.8.0-35-generic
Kata guest kernel: WARN[0000] cannot set cgroup manager to "systemd" for runtime "io.containerd.kata.v2" 
Linux version 6.1.62 (root@25cbaeba2362) (gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #1 SMP Tue Mar  5 10:00:02 UTC 2024

```
- Host (runc): 6.8.0-35-generic
- Kata: 6.1.62

- Kata uses a separate guest kernel (VM-based isolation)

**Compare CPU models (real vs virtualized)**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== CPU Model Comparison ===" | tee labs/lab12/analysis/cpu-comparison.txt

echo "Host CPU:" | tee -a labs/lab12/analysis/cpu-comparison.txt
grep "model name" /proc/cpuinfo | head -1 | tee -a labs/lab12/analysis/cpu-comparison.txt

echo "Kata VM CPU:" | tee -a labs/lab12/analysis/cpu-comparison.txt
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 sh -c "grep 'model name' /proc/cpuinfo | head -1" | tee -a labs/lab12/analysis/cpu-comparison.txt
=== CPU Model Comparison ===
Host CPU:
model name	: Intel Xeon Processor (Skylake, IBRS)
Kata VM CPU:
model name	: Intel Xeon Processor (Skylake, IBRS)
```
- Host CPU: Intel Xeon Processor (Skylake, IBRS)
- Kata CPU: Intel Xeon Processor (Skylake, IBRS)

- CPU appears similar but is virtualized inside Kata VM

**Isolation Analysis**
- **runc**:
  - Shares host kernel
  - Uses namespaces and cgroups
  - Weaker isolation

- **Kata**:
  - Runs each container inside a lightweight VM
  - Separate kernel and virtualized hardware
  - Strong isolation boundary


## Task 3

**Show dmesg output differences (Kata shows VM boot logs, proving separate kernel)**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# mkdir -p labs/lab12/isolation

echo "=== dmesg Access Test ===" | tee labs/lab12/isolation/dmesg.txt

echo "Kata VM (separate kernel boot logs):" | tee -a labs/lab12/isolation/dmesg.txt  
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 dmesg 2>&1 | head -5 | tee -a labs/lab12/isolation/dmesg.txt
=== dmesg Access Test ===
Kata VM (separate kernel boot logs):
time="2026-04-20T12:52:50Z" level=warning msg="cannot set cgroup manager to \"systemd\" for runtime \"io.containerd.kata.v2\""
[    0.000000] Linux version 6.1.62 (root@25cbaeba2362) (gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #1 SMP Tue Mar  5 10:00:02 UTC 2024
[    0.000000] Command line: tsc=reliable no_timer_check rcupdate.rcu_expedited=1 i8042.direct=1 i8042.dumbkbd=1 i8042.nopnp=1 i8042.noaux=1 noreplace-smp reboot=k cryptomgr.notests net.ifnames=0 pci=lastbus=0 root=/dev/pmem0p1 rootflags=dax,data=ordered,errors=remount-ro ro rootfstype=ext4 console=hvc0 console=hvc1 quiet systemd.show_status=false panic=1 nr_cpus=2 selinux=0 systemd.unit=kata-containers.target systemd.mask=systemd-networkd.service systemd.mask=systemd-networkd.socket scsi_mod.scan=none
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
```

**Compare /proc filesystem visibility**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== /proc Entries Count ===" | tee labs/lab12/isolation/proc.txt

echo -n "Host: " | tee -a labs/lab12/isolation/proc.txt
ls /proc | wc -l | tee -a labs/lab12/isolation/proc.txt

echo -n "Kata VM: " | tee -a labs/lab12/isolation/proc.txt
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 sh -c "ls /proc | wc -l" | tee -a labs/lab12/isolation/proc.txt
=== /proc Entries Count ===
Host: 176
52
```

- Host: 176 entries
- Kata: 52 entries

- Kata VM sees fewer processes (isolated environment)

**Show network interface configuration in Kata VM** 
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== Network Interfaces ===" | tee labs/lab12/isolation/network.txt

echo "Kata VM network:" | tee -a labs/lab12/isolation/network.txt
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 ip addr | tee -a labs/lab12/isolation/network.txt
=== Network Interfaces ===
Kata VM network:
WARN[0000] cannot set cgroup manager to "systemd" for runtime "io.containerd.kata.v2" 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP qlen 1000
    link/ether a2:85:5c:62:74:6a brd ff:ff:ff:ff:ff:ff
    inet 10.4.0.11/24 brd 10.4.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a085:5cff:fe62:746a/64 scope link tentative 
       valid_lft forever preferred_lft forever
```
- Separate virtual network environment

**Compare kernel module counts (host vs guest VM)**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== Kernel Modules Count ===" | tee labs/lab12/isolation/modules.txt

echo -n "Host kernel modules: " | tee -a labs/lab12/isolation/modules.txt
ls /sys/module | wc -l | tee -a labs/lab12/isolation/modules.txt

echo -n "Kata guest kernel modules: " | tee -a labs/lab12/isolation/modules.txt
sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 sh -c "ls /sys/module 2>/dev/null | wc -l" | tee -a labs/lab12/isolation/modules.txt
=== Kernel Modules Count ===
Host kernel modules: 192
67
```
- Host: 192 modules
- Kata: 67 modules
- Kata uses minimal guest kernel

**Isolation Analysis**
- **runc**:
  - Shares host kernel and modules
  - Limited isolation via namespaces/cgroups
  - Access to host-level resources possible

- **Kata**:
  - Runs in a dedicated VM
  - Separate kernel, modules, and network stack
  - Strong isolation boundary


**Security Implications**
- **runc**:
  - Container escape = potential host compromise
  - Vulnerabilities affect entire system

- **Kata**:
  - Container escape = limited to VM
  - Host remains protected

## Task 4

**Show startup time comparison (runc: <1s, Kata: 3-5s)**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== Startup Time Comparison ===" | tee labs/lab12/bench/startup.txt

echo "runc:" | tee -a labs/lab12/bench/startup.txt
time sudo nerdctl run --rm alpine:3.19 echo "test" 2>&1 | grep real | tee -a labs/lab12/bench/startup.txt

echo "Kata:" | tee -a labs/lab12/bench/startup.txt
time sudo nerdctl run --rm --runtime io.containerd.kata.v2 alpine:3.19 echo "test" 2>&1 | grep real | tee -a labs/lab12/bench/startup.txt
=== Startup Time Comparison ===
runc:

real	0m1.553s
user	0m0.013s
sys	0m0.033s
Kata:

real	0m4.574s
user	0m0.013s
sys	0m0.021s
```

**Show HTTP latency for juice-runc baseline**
```
root@Server-fc9c1449-6bff-41e8-94c7-5b4b13e75ada:~# echo "=== HTTP Latency Test (juice-runc) ===" | tee labs/lab12/bench/http-latency.txt

out="labs/lab12/bench/curl-3012.txt"
: > "$out"

for i in $(seq 1 50); do
  curl -s -o /dev/null -w "%{time_total}\n" http://localhost:3012/ >> "$out"
done

echo "Results for port 3012 (juice-runc):" | tee -a labs/lab12/bench/http-latency.txt

awk '{s+=$1; n+=1} END {if(n>0) printf "avg=%.4fs min=%.4fs max=%.4fs n=%d\n", s/n, min, max, n}' \
  min=$(sort -n "$out" | head -1) max=$(sort -n "$out" | tail -1) "$out" | tee -a labs/lab12/bench/http-latency.txt
=== HTTP Latency Test (juice-runc) ===
Results for port 3012 (juice-runc):
avg=0.0103s min=0.0040s max=0.0699s n=50
```

**Performance Analysis**

- **Startup overhead**:
  - runc: minimal (container only)
  - Kata: high (VM boot required)

- **Runtime overhead**:
  - runc: near-native performance
  - Kata: slight overhead due to virtualization

- **CPU overhead**:
  - runc: direct access to host CPU
  - Kata: virtualized CPU adds overhead


**When to use each**
- **Use runc when**:
  - performance is critical
  - low latency is required
  - trusted workloads

- **Use Kata when**:
  - strong isolation is required
  - running untrusted code
  - multi-tenant environments