Can you give me the full example of heartbeat 8.10 selinux security policy on CentOS 8?



Sure! Below is a full example of a custom SELinux security policy for **Elastic Heartbeat 8.10** on **CentOS 8**. This policy allows Heartbeat to run without encountering SELinux denials related to typical interactions such as reading/writing to files, connecting over the network, and interacting with processes and services. I've included detailed comments to help you understand each part.

### 1. **Install Required Tools**
First, ensure you have the necessary tools to create and manage SELinux policies:
```bash
sudo dnf install policycoreutils-devel selinux-policy-devel
```

### 2. **Create the SELinux Policy Directory**
Create a directory to store your SELinux policy files:
```bash
mkdir ~/heartbeat-selinux
cd ~/heartbeat-selinux
```

### 3. **Create the SELinux Policy File (`heartbeat.te`)**
Create the file `heartbeat.te` to define the security policy for Elastic Heartbeat:
```bash
nano heartbeat.te
```

### Full Example of `heartbeat.te`:
```bash
module heartbeat 1.0;

require {
    type unconfined_t;          # The type for unconfined processes
    type syslogd_t;             # The type for the syslog daemon
    type heartbeat_t;           # The type for the Heartbeat process (you will define this)
    class file { read write execute };  # File access classes
    class process { transition };       # Process transition rules
    class tcp_socket { bind listen connect accept };  # TCP socket permissions
    class udp_socket { bind listen connect accept };  # UDP socket permissions
}

# Allow heartbeat to read, write, and execute files in its directory
allow heartbeat_t syslogd_t:file { read write execute };

# Allow heartbeat to transition to unconfined processes
allow heartbeat_t unconfined_t:process transition;

# Allow heartbeat to communicate via TCP and UDP
allow heartbeat_t self:tcp_socket { bind listen connect accept };
allow heartbeat_t self:udp_socket { bind listen connect accept };

# Allow heartbeat to read and write log files
allow heartbeat_t syslogd_t:file { read write };

# Allow heartbeat to communicate with the syslog daemon
allow heartbeat_t syslogd_t:process { transition };

# Allow heartbeat to execute binaries
allow heartbeat_t self:process execmod;
```

### Explanation of the Policy:
- **Types:**
  - `heartbeat_t`: The type for the Heartbeat service.
  - `unconfined_t`: Used for processes running in an unconfined environment (like systemd or the root user).
  - `syslogd_t`: The type for syslog daemon processes.
  
- **Allowing File Access:**
  - The policy allows Heartbeat (`heartbeat_t`) to read, write, and execute files associated with the syslog daemon (`syslogd_t`).
  
- **Network Access:**
  - It grants Heartbeat the ability to bind, listen, connect, and accept on both TCP and UDP sockets.

- **Process Transition:**
  - Heartbeat is allowed to transition to unconfined processes, allowing it to perform operations outside its security domain, which may be required for specific system tasks.
  
- **Log File Access:**
  - The policy allows Heartbeat to read/write to syslog log files.

- **Execute Permission:**
  - It grants Heartbeat permission to execute binaries (using `execmod`).

### 4. **Compile the Policy**
Once you've defined your policy file, you need to compile it into a loadable SELinux module.

- Compile the `.te` file into a `.mod` file:
  ```bash
  checkmodule -M -m -o heartbeat.mod heartbeat.te
  ```

- Create a package from the compiled module:
  ```bash
  semodule_package -o heartbeat.pp -m heartbeat.mod
  ```

### 5. **Install the SELinux Policy Module**
Now install the SELinux module you just created:
```bash
sudo semodule -i heartbeat.pp
```

### 6. **Verify SELinux Status**
You can check the status of SELinux and ensure that the module is installed correctly by running:
```bash
sestatus
```

### 7. **Test and Monitor Logs**
Test the Heartbeat service to ensure it is working without SELinux interference. If Heartbeat is still being blocked or denied by SELinux, you can view the relevant log entries using:

```bash
ausearch -m avc -ts recent
```

This will display any recent SELinux denials. If any specific denials related to Heartbeat show up, you can either adjust the policy or generate additional rules using `audit2allow`.

### Example of using `audit2allow`:
If SELinux denials are observed, you can generate and apply additional permissions by running:

```bash
ausearch -m avc -ts recent | audit2allow -M heartbeat_custom
sudo semodule -i heartbeat_custom.pp
```

### 8. **Monitor SELinux Logs**
You can continue to monitor SELinux logs using:
```bash
journalctl -xe | grep SELinux
```

This allows you to catch any issues as they arise.

---

### Final Thoughts:
This example SELinux policy for Elastic Heartbeat 8.10 on CentOS 8 should allow the service to function without interference from SELinux while maintaining the security posture of the system. Keep in mind that this is just a basic example, and depending on your exact configuration and use case, you may need to adjust the policy further.

If you run into specific SELinux denials, the `audit2allow` tool will be very helpful in generating the exact permissions you need to allow Heartbeat to function properly.