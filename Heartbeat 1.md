How to make elastic heartbeat 8.10 selinux security policy on CentOS 8



To make an SELinux security policy for Elastic Heartbeat 8.10 on CentOS 8, you need to configure a custom SELinux module that allows Heartbeat to work without being blocked by SELinux. Below are the steps to create and install a custom SELinux policy for Elastic Heartbeat 8.10.

### Steps:

#### 1. **Install the required tools:**

Make sure you have the necessary tools installed to manage SELinux policies. If not, you can install them using the following command:

```bash
sudo dnf install policycoreutils-devel selinux-policy-devel
```

#### 2. **Create a custom SELinux policy file:**

You'll need to create a custom SELinux module to allow the Elastic Heartbeat 8.10 service to work. Start by creating a directory to store the policy file:

```bash
mkdir ~/heartbeat-selinux
cd ~/heartbeat-selinux
```

Create a policy file named `heartbeat.te` (or any name you prefer):

```bash
nano heartbeat.te
```

In this file, you need to define the custom policy rules. A simple policy might look like this:

```bash
module heartbeat 1.0;

require {
    type unconfined_t;
    type syslogd_t;
    class file { read write execute };
    class process { transition };
}

# Allow heartbeat to read and write files in its directory
allow heartbeat_t syslogd_t:file { read write execute };
allow heartbeat_t unconfined_t:process transition;

# Allow heartbeat to communicate via network
allow heartbeat_t self:tcp_socket { bind listen connect accept };
allow heartbeat_t self:udp_socket { bind listen connect accept };

# Allow heartbeat to manage syslog
allow heartbeat_t syslogd_t:process { transition };
```

This is just an example, and you may need to adjust the policy based on the specific needs and actions of Heartbeat 8.10.

#### 3. **Generate the policy module:**

After creating the policy file, you can now compile it into a loadable SELinux module:

```bash
checkmodule -M -m -o heartbeat.mod heartbeat.te
semodule_package -o heartbeat.pp -m heartbeat.mod
```

#### 4. **Install the SELinux policy module:**

Now, install the newly created SELinux policy module:

```bash
sudo semodule -i heartbeat.pp
```

#### 5. **Verify the policy:**

To check if the policy was successfully installed, run:

```bash
sestatus
```

You should see that your custom policy is now in place and that Elastic Heartbeat is allowed to operate without SELinux blocking it.

#### 6. **Troubleshooting:**

If you run into issues where Elastic Heartbeat is still being blocked, you can check the SELinux logs for any denials using `audit2why` and `audit2allow`:

```bash
ausearch -m avc -ts recent
```

This will show you the recent denials. You can then generate additional policy rules to fix any issues using `audit2allow`.

For example:

```bash
ausearch -m avc -ts recent | audit2allow -M heartbeat_custom
sudo semodule -i heartbeat_custom.pp
```

#### 7. **Test and Monitor:**

Finally, make sure to test Heartbeat and monitor any new SELinux denials. If everything works fine, Elastic Heartbeat should now operate smoothly on CentOS 8 with SELinux enabled.

### Conclusion

Creating an SELinux policy for Elastic Heartbeat on CentOS 8 involves defining the required permissions in a custom SELinux policy file and then compiling and installing it. You may need to adjust the policy based on your specific configuration and requirements.