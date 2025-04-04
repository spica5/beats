How to allow Elastic Metricbeat to access audit log on CentOS 8 selinux is on?



To allow **Elastic Metricbeat** to access the **audit log** on **CentOS 8** when SELinux is enabled, you need to create and apply a custom SELinux policy. By default, SELinux restricts access to certain files, and Metricbeat may not have the necessary permissions to read the audit log files.

Here are the steps to create an SELinux policy that grants Metricbeat access to the audit logs:

### 1. **Install Required Tools**

Ensure that you have the necessary tools installed to manage SELinux policies:

```bash
sudo dnf install policycoreutils-devel selinux-policy-devel
```

### 2. **Identify the Audit Log File Path**

By default, the audit log files are located in `/var/log/audit/`, and the main audit log file is usually named `audit.log`. You should verify the location of the audit logs on your system.

To check the location of your audit logs, you can run:

```bash
ls /var/log/audit/
```

Make a note of the exact path to the log file (typically `/var/log/audit/audit.log`).

### 3. **Create a Custom SELinux Policy File**

Now, create a custom SELinux policy file to allow Metricbeat to read the audit log files.

- First, create a directory to store your SELinux policy:

  ```bash
  mkdir ~/metricbeat-selinux
  cd ~/metricbeat-selinux
  ```

- Next, create the policy file `metricbeat_audit_log.te`:

  ```bash
  nano metricbeat_audit_log.te
  ```

Add the following content to the file:

```bash
module metricbeat_audit_log 1.0;

require {
    type unconfined_t;             # Unconfined processes (e.g., Metricbeat)
    type auditd_t;                 # Audit daemon type
    type metricbeat_t;             # Metricbeat process type
    class file { read };           # File read permissions
}

# Allow Metricbeat to read the audit log files
allow metricbeat_t auditd_t:file { read };
```

### Explanation of the Policy:

- **Types:**
  - `metricbeat_t`: This is the type for the Metricbeat process.
  - `auditd_t`: This is the type for audit-related processes.
  
- **Allow Access:**
  - The policy grants the Metricbeat process (`metricbeat_t`) permission to read files that belong to the `auditd_t` type (i.e., audit log files).

### 4. **Compile the SELinux Policy**

Once the policy is written, you need to compile it into a loadable SELinux module.

- First, compile the `.te` file into a `.mod` file:

  ```bash
  checkmodule -M -m -o metricbeat_audit_log.mod metricbeat_audit_log.te
  ```

- Then, create the SELinux package (`.pp` file):

  ```bash
  semodule_package -o metricbeat_audit_log.pp -m metricbeat_audit_log.mod
  ```

### 5. **Install the SELinux Policy Module**

Install the SELinux module you just created:

```bash
sudo semodule -i metricbeat_audit_log.pp
```

### 6. **Verify the SELinux Status**

Ensure that the SELinux module was installed successfully by checking the SELinux status:

```bash
sestatus
```

### 7. **Test Metricbeat Access to the Audit Log**

Now, start Metricbeat and verify that it can access the audit log without encountering SELinux denials.

You can check if Metricbeat encounters any SELinux denials by looking at the SELinux logs:

```bash
ausearch -m avc -ts recent
```

If you see any denials related to Metricbeat accessing the audit log, you can troubleshoot by running `audit2allow` to generate additional rules or review the specific denials.

### 8. **Monitor Logs (Optional)**

To monitor any further SELinux denials in real-time, use:

```bash
journalctl -xe | grep SELinux
```

This will help you identify any future issues with access permissions.

---

### Conclusion

By following these steps, you've created a custom SELinux policy that allows **Elastic Metricbeat** to access the **audit log** on a **CentOS 8** system. This approach ensures that Metricbeat has the necessary permissions to read the audit logs while maintaining the security restrictions enforced by SELinux.

If you run into further issues, the `ausearch` and `audit2allow` tools are very helpful for identifying and resolving SELinux denials.