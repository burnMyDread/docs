## Overview:

With Systemd version 245 and latter sytemd introduced a new tool, systemd-analyze security, to audit the security of systemd service files. 
This document will guide you through the results of these audits.

## Key controls

This section describes the main controls systemd exposes to you as the service aurthor. Note: That in systemd syntax, ~ means not.

### Syscalls

In order to get any work done, a program needs to ask the OS to actully use the machine. This access is mediated via syscalls.
Systemd provides both whitelisting and backlisting capabilities. 
In general, the best practice is to only whitelist the syscalls you need. A good starting point is @system-service.
However as an admin or devloper you many not know all the syscalls you need, in that case blacklisting known dangerious syscalls is a good option.

These syscalls should be avoided: 
- @clock
- @debug
- @module
- @mount
- @obsolete
- @privileged
- @raw-io
- @reboot

Note: Setting the SystemCallFilter to ~<syscall group> will blacklist the group.

[source](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
  
### Capabilities 

Capabilities are collections of syscalls. 
These collections don't always line up with systemd's syscall groups therefor if you are blacklisting system calls you should also blacklist dangerous capabilities.

Note: Not all are the same size, cap_sys_admin and cap_net_admin have very broad access to syscalls and should not be used without serious descution with your local security advisor.
  
Capabilities to avoid:
- CAP_SYS_ADMIN
- CAP_NET_ADMIN
- CAP_NET_RAW
- CAP_DAC_OVERIDE
- CAP_MAC_OVERIDE
- CAP_MAC_ADMIN
- CAP_PTRACE
- CAP_SYS_RAWIO
- CAP_SYS_TIME
  
[Source](https://man7.org/linux/man-pages/man7/capabilities.7.html)
  
### Filesystem Isolation
Systemd has a few diffrent tools for isolating the host files from the services files. Most of these are covered by the protect<Something> bit not all of them.
This is where you should put the pricpal of least privilage in full force.
  
Note: This documentassumes you have a working knowlage of [kernel pseudo filesystems](https://medium.com/@jain.sm/pseudo-file-systems-in-linux-5bf67eb6e450) and [mounts](https://www.computerhope.com/unix/umount.htm)
  
- ProtectSystem
    Takes a boolean argument or the special values "full" or "strict". If true, mounts the /usr/ and the boot loader directories (/boot and /efi) read-only for processes invoked by this unit. If set to "full", the /etc/ directory is mounted read-only, too. If set to "strict" the entire file system hierarchy is mounted read-only, except for the API file system subtrees /dev/, /proc/ and /sys/ (protect these directories using PrivateDevices=, ProtectKernelTunables=, ProtectControlGroups=). (Note: This is from the linked source.)
- ProtectProc
    This limits access to the golbal proc peudofilesystem which hash process information for the whole system by default. This can be limied to just the process table for the process.
- ProtectHome
    This will make user home accounts show up as empty. This relies on umount not being avalible to the process.

[source](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
  
### Process Isolation
  
- Namespace isolation
  Namespaces allow limiting what processes can view the other processes in a namespace via kernel options. This means that if a process does ps or tries to ptrace it will only see processes in its namespace.
- LimitX
  This sets ulimits which are not set by default except for user processes. (In relatiy this is anything that sources the shell profile.)
- ProtectProc
  This limits access to the default proc filesystem. This is useful in that it limits the process information that a process can access. This is useful in the container space as it limits what a contianer can do in the case of a breakout.
- ReadWritePaths, ReadOnlyPaths, InaccessiblePaths, ExecPaths, NoExecPaths
  These allow for fine grained access controls beyond Descretionary Access Controls. You can use these to further spesifiy what access a process has on the host filesystem.
- ProtectClock
  This protects the system clock from being set by the process. If this is unset or set to false a malicous process can change the system time to break tls.
- ProtectKernelModules
  This stops the process from loading kernel modules. This should almost always be set to true.
- MemoryDenyWriteExecute
  This is a bufferoverflow protection. This should almost always be set to true.

[source](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)
  
### Misc.

- NoNewPrivileges
  This blocks processes from becoming root.
- SELinuxContext, AppArmorProfile, SmackProcessLabel
  These force the process into the coresponding Mandiory Access Control label. This can be used to either further isolate a process or disable the labeling in a targeted fashion.
- KeyringMode
  This is an isolation feature for the system keyring, the two main usecases are: If you want your process to keep its keys out of the system keyring. (Becuase you are doing something sensitive.) Or you are running an untrusted or less sensitive process on a host with sensitive processes.
  
