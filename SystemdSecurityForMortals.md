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

Note: I haven't found a list of all the systemd syscall groups yet, if you find one let me know.  
  
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

### Process Isolation
  
### Misc.
