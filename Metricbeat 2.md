Can you give me the full example of Metricbeat 8.10 selinux security policy on CentOS 8?


Sure! Below is a full example of an SELinux security policy for **Metricbeat 8.10** on **CentOS 8**, which will allow Metricbeat to access necessary resources, such as audit logs and other files, while SELinux is enabled.

### 1. **Install Required Tools**

First, install the necessary SELinux tools on CentOS 8, if not already installed:

```bash
sudo dnf install policycoreutils-devel selinux-policy-devel
```

### 2. **Create the SELinux Policy Directory**

Create a directory to store your SELinux policy files:

```bash
mkdir ~/metricbeat-selinux
cd ~/metricbeat-selinux
```

### 3. **Create the SELinux Policy File for Metricbeat**

Create the file `metricbeat.te` to define the security policy for Metricbeat 8.10:

```bash
nano metricbeat.te
```

### Full Example of `metricbeat.te`:

```bash
module metricbeat 1.0;

require {
    type unconfined_t;             # Unconfined processes (e.g., Metricbeat)
    type auditd_t;                 # Audit daemon type
    type metricbeat_t;             # Metricbeat process type
    class file { read write execute };  # File permissions: read, write, execute
    class process { transition };      # Process permissions: transition
    class tcp_socket { bind listen connect accept };  # TCP socket permissions
    class udp_socket { bind listen connect accept };  # UDP socket permissions
}

# Allow Metricbeat to read the audit logs (default location: /var/log/audit/)
allow metricbeat_t auditd_t:file { read };

# Allow Metricbeat to read files in /etc/metricbeat and other Metricbeat directories
allow metricbeat_t etc_t:file { read execute };

# Allow Metricbeat to write to its own directories (e.g., for storing state files)
allow metricbeat_t var_lib_t:file { write create };

# Allow Metricbeat to communicate via TCP and UDP sockets
allow metricbeat_t self:tcp_socket { bind listen connect accept };
allow metricbeat_t self:udp_socket { bind listen connect accept };

# Allow Metricbeat to execute binaries
allow metricbeat_t self:process execmod;

# Allow Metricbeat to transition to unconfined_t (if necessary)
allow metricbeat_t unconfined_t:process transition;

# Allow Metricbeat to read from syslog
allow metricbeat_t syslogd_t:file { read };
```

### Explanation of the Policy:

1. **Types:**
   - `metricbeat_t`: The type associated with the Metricbeat process.
   - `auditd_t`: The type for the audit daemon, which owns the audit logs.
   - `unconfined_t`: A type for processes that can operate outside SELinux's constraints (used for things like systemd).
   - `etc_t` and `var_lib_t`: Types for directories where Metricbeat needs read/write access.

2. **Permissions:**
   - **File permissions:**
     - The policy allows Metricbeat to read the audit logs (`/var/log/audit/audit.log`) by allowing `metricbeat_t` to read files of type `auditd_t`.
     - It also allows Metricbeat to read files in its own configuration directory (`/etc/metricbeat/`) and its state directory (`/var/lib/metricbeat/`).
   - **Socket permissions:**
     - Metricbeat can bind, listen, connect, and accept TCP and UDP connections, which is often required for monitoring and gathering metrics over the network.
   - **Process permissions:**
     - Metricbeat is allowed to execute binaries and transition to processes of the `unconfined_t` type if necessary.

### 4. **Compile the SELinux Policy**

After writing the policy file, compile it into a SELinux module.

- Compile the `.te` file into a `.mod` file:

  ```bash
  checkmodule -M -m -o metricbeat.mod metricbeat.te
  ```

- Then, create the SELinux policy package (`.pp` file):

  ```bash
  semodule_package -o metricbeat.pp -m metricbeat.mod
  ```

### 5. **Install the SELinux Policy Module**

Install the SELinux module:

```bash
sudo semodule -i metricbeat.pp
```

### 6. **Verify SELinux Status**

Check the SELinux status and ensure the policy module was installed successfully:

```bash
sestatus
```

Ensure that the SELinux status indicates the system is enforcing policies.

### 7. **Test Metricbeat Access**

Now, test Metricbeat to ensure it can access the audit logs and other necessary files. If Metricbeat encounters any SELinux denials, you can use `ausearch` to review the logs.

For example, to view recent denials:

```bash
ausearch -m avc -ts recent
```

### 8. **Troubleshooting with `audit2allow`**

If you observe SELinux denials, you can use the `audit2allow` tool to automatically generate additional SELinux policy rules to allow the denied actions. Here’s an example:

1. Check for denials:

   ```bash
   ausearch -m avc -ts recent
   ```

2. Use `audit2allow` to create a new policy module to allow the denials:

   ```bash
   ausearch -m avc -ts recent | audit2allow -M metricbeat_custom
   ```

3. Install the newly generated module:

   ```bash
   sudo semodule -i metricbeat_custom.pp
   ```

### 9. **Monitor Logs (Optional)**

You can continue to monitor the SELinux logs to catch any future denials by using:

```bash
journalctl -xe | grep SELinux
```

This command will display any SELinux-related issues and help you troubleshoot any further problems.

---

### Conclusion

This SELinux policy example for **Metricbeat 8.10** on **CentOS 8** allows Metricbeat to access the audit logs and perform necessary actions while maintaining SELinux security constraints. The policy grants permission for file access, network communication, and process transitions. 

If Metricbeat still encounters issues, you can adjust the policy by checking the SELinux audit logs and using `audit2allow` to create additional rules.