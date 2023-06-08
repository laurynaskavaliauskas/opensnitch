General
---

**OpenSnitch displays too many dialogs to allow/deny connections**

Yes, it does. But only the first time it is used. Once you configure which processes/connections you want to allow/deny, you won't notice that it's running. Really.

In the future, maybe we can add an option to queue events, and allow/deny them from the GUI: applying the default configured action until further interaction from the user.


**Why Qt and not GTK?**

I tried, but for very fast updates it failed bad on my configuration (failed bad = SIGSEGV), moreover I find Qt5 layout system superior and easier to use.


**Why gRPC and not DBUS?**

The UI service is able to use a TCP listener instead of a UNIX socket, that means the UI service itself can be executed on any operating system, while receiving messages from a single local daemon instance or multiple instances from remote computers in the network, therefore DBUS would have made the protocol and logic uselessly GNU/Linux specific.

Connections
---

**Status is Not Running** message on the GUI

Be sure that the daemon is running: `$ pgrep opensnitchd`

If it's not running, you may need to enable and start it: 

```
$ sudo systemctl enable opensnitchd
$ sudo systemctl start opensnitchd.service 
```

**Why is WireGuard/Mullvad/etc not working with OpenSnitch?**

The common reason is because the eBPF module is not installed or not working.

If you are using the packages from AUR, you need to install the additional package https://aur.archlinux.org/packages/opensnitch-ebpf-module

Another reason could be , that the eBPF module insertion failed or the debugfs is not mounted. In both cases, execute the following command:

`$ sudo cat /sys/kernel/debug/tracing/kprobe_event`

If the output is empty or it fails, then you can try mounting it: `# mount -t debugfs none /sys/kernel/debug` then restart opensnitch: `sudo service opensnitch restart`

If it still doesn't work, you can enable `[x] Debug invalid connections` under Preferences->Nodes.

[More info](https://github.com/evilsocket/opensnitch/tree/master/ebpf_prog)

**Which connections does OpenSnitch intercept?**

We currently (>= v1.6.0-rc.4) intercept new connections (iptables/conntrack state NEW) of TCP, UDP and UDPLITE, SCTP and ICMP protocols, to/from any port.

Kernels support
---

Your kernel needs some features to be enabled in order eBPF to work: debugfs (or tracefs), kprobes, perf events, ftrace and syscalls (bpf and ftrace).

You can check if they're available for your kernel with the following commands:

```bash
 $ grep -E "(BPF|FTRACE|KPROBE)" /boot/config-$(uname -r)
  CONFIG_CGROUP_BPF=y
  CONFIG_BPF=y
  CONFIG_BPF_SYSCALL=y
  CONFIG_BPF_EVENTS=y
  CONFIG_KPROBES=y
  CONFIG_KPROBE_EVENTS=y
  CONFIG_FTRACE=y
  CONFIG_FTRACE_SYSCALLS=y
```
If any of the above options appears as "is not set", your kernel lacks support for it.

```bash
$ sudo ls /sys/kernel/debug/tracing/kprobe_events
$ sudo ls /sys/kernel/debug/tracing/events/syscalls/
```

If some of the above commands outputs "no such file or directory", your kernel lacks support for it.

[More info](https://github.com/evilsocket/opensnitch/tree/master/ebpf_prog)

**xanmod and liquorix kernels**

Unfortunately, these kernels are compiled without the mentioned features, so eBPF process monitor method won't be available on these kernels.
liquorix does have support for kprobes, but no syscalls tracing. But xanmod doesn't have support for any of the needed features.

For these kernels, the default method to intercept processes will be ProcFS (proc).

**hardened kernels**

We support most of the kernel hardening options. However some of them causes eBPF not to work. We don't know yet (18/11/2022) which is exactly the option that prevent us to work as expected.

Configuration
---

**What does the "Debug invalid connections" configuration option mean?**

When a process establishes a new connection, we first receive the connection information (src/dst IP, src/dst port, but no PID, nor process command line/path). Thus, we try to get who created the connection.

Sometimes we fail to discover the PID of the process, or the path of the PID, thus in these cases if you check this option, a pop-up will appear to allow or deny an "unknown connection".

**What's the behaviour of daemon's default action "deny"**

The daemon option "default_action" "deny" will block ALL traffic (as of version 1.6.0-rc.4) that is intercepted by _iptables_ or _nftables_ and is not answered or configured by the user. If an outgoing connection timeouts while waiting for user action, then it'll apply the default action.

If you suspect that opensnitch is blocking an application and asking you to allow/deny it (for example VPN traffic), enable the option `[x] Debug invalid connections` from Preferences -> Nodes

If you need to allow this kind of traffic, you can add a rule directly to iptables/nftables:

`iptables -t mangle -I OUTPUT -p icmp -j ACCEPT`

Read more 👉 https://github.com/evilsocket/opensnitch/wiki/System-rules-legacy

Rules
---

**In which order does opensnitch check configured rules?**

Since version 1.2.0, rules are checked in alphabetical order. There's also a new field to mark a rule as Important.

So if you want to prioritize some rules over others:
1. Name the rule as 000-max-priority, 001-notsomax-priority, 002-less-preiority, not-priority
2. [x] Priority field checked (Action: allow)
3. OR Action: deny (not need to check the Priority field in these rules)

**If I allow program A, and it launches another program B, will it be also allowed?**

No. You only allow program A to access the net. Any other program launched by program A will be stopped until you allow or deny it.

See some examples: 
 - Spotify launching wget: https://github.com/evilsocket/opensnitch/discussions/401
 - Vivaldi browser deb package trying to install from the internet additional packages: https://github.com/evilsocket/opensnitch/discussions/742

Read more about best practices: https://github.com/evilsocket/opensnitch/wiki/Rules#best-practices

**Can a malicious program simply open another process or port and bypass an application based filter?**

A process may open other subprocesses, but will it bypass defined application rules? No (see previous FAQ why ^). From the OpenSnitch perspective, it'll just be a new process opening an outbound connection without a rule defined for it, and as such, it'll ask you to allow or deny it.

See examples of malware running on GNU/Linux and how OpenSnitch detects the outbound connections:

https://github.com/evilsocket/opensnitch/discussions/791
https://github.com/evilsocket/opensnitch/discussions/743
https://github.com/evilsocket/opensnitch/discussions/742
https://github.com/evilsocket/opensnitch/discussions/564

If you create a rule to allow `wget` or `curl` system-wide, a malicious process may use of it to download remote files, so it all depends on what rules you define:

https://github.com/evilsocket/opensnitch/wiki/Rules#best-practices

Anyway, nothing is unbreakable. If you know a way to bypass application rules, we'd love to see a detailed example! That'll help us to improve the application.

**Appimages confuse the firewall**

Appimages create a random directory under `/tmp/` from where they're executed, so if you allow or deny an appimage by path or command line when the pop-up appears, the next time the app is executed, the path to the binary will be different and OpenSnitch will prompt you again to deny or allow it.

You need to use regular expressions to match the directory by editing the rule:

[x] From this executable: ^(/tmp/\.mount_Archiv[0-9A-Za-z]+/.*)$

See this issue for context and more information: [#408](https://github.com/evilsocket/opensnitch/issues/408)

Other
---

**Do I need to turn off or uninstall other firewalling (firewalld, ufw, gufw) before installing OpenSnitch ? Will the OpenSnitch install or app turn them off automatically ?**

No, you don't need to turn off or uninstall other firewalling. OpenSnitch doesn't turn them off, nor delete their rules.

[Read more](https://github.com/evilsocket/opensnitch/wiki/Dependencies-and-how-it-works)