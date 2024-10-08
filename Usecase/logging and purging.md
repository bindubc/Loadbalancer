To set up logging and log rotation on a Linode server using HAProxy with two load balancers, follow these steps:

### Step 1: Configure HAProxy to Enable Logging
HAProxy uses the `rsyslog` service to manage logs. We will configure HAProxy to send logs to `rsyslog` and set up log rotation for purging logs older than one day.

1. **Enable HAProxy Logging:**
   Open the HAProxy configuration file, usually located at `/etc/haproxy/haproxy.cfg`:

   ```bash
   sudo nano /etc/haproxy/haproxy.cfg
   ```

2. **Modify the Configuration:**
   Add the following lines at the top of the file (before `global` and `defaults` sections) to enable logging via `rsyslog`:

   ```haproxy
   global
       log /dev/log local0
       log /dev/log local1 notice
       log 127.0.0.1:514 local0
       maxconn 4096

   defaults
       log     global
       mode    http
       option  httplog
       option  dontlognull
       timeout connect 5000ms
       timeout client  50000ms
       timeout server  50000ms
   ```

   These configurations will log both standard and notice-level messages.

3. **Update `rsyslog` Configuration:**
   By default, HAProxy may not be set up to send logs to `rsyslog`. Enable it by editing `/etc/rsyslog.d/haproxy.conf`:

   ```bash
   sudo nano /etc/rsyslog.d/haproxy.conf
   ```

   Add the following lines:

   ```rsyslog
   # Enable HAProxy logging
   $ModLoad imudp
   $UDPServerRun 514

   local0.*    /var/log/haproxy.log
   ```

4. **Restart HAProxy and rsyslog:**
   After making changes, restart both services to apply the configurations:

   ```bash
   sudo systemctl restart rsyslog
   sudo systemctl restart haproxy
   ```

### Step 2: Verify Logging
Check if logs are being generated correctly:

```bash
tail -f /var/log/haproxy.log
```

### Step 3: Set Up Log Rotation with `logrotate`
To ensure logs are purged after one day, configure `logrotate`. This will automatically delete older logs, keeping only the last 24 hours of entries.

1. **Create a Logrotate Configuration File:**

   Create a new configuration file for HAProxy in `/etc/logrotate.d/`:

   ```bash
   sudo nano /etc/logrotate.d/haproxy
   ```

2. **Add the Log Rotation Settings:**

   Add the following content to configure log rotation:

   ```bash
   /var/log/haproxy.log {
       daily
       missingok
       rotate 1
       compress
       delaycompress
       notifempty
       create 640 root adm
       postrotate
           /usr/bin/systemctl reload haproxy > /dev/null 2>/dev/null || true
       endscript
   }
   ```

   Explanation:
   - `daily` – Rotate logs every day.
   - `rotate 1` – Keep only 1 rotated log file (older logs will be deleted).
   - `compress` – Compress the log file after rotation.
   - `delaycompress` – Delay compression to avoid issues with reloading.
   - `notifempty` – Skip rotation if the log is empty.
   - `postrotate` – Command to run after rotation (reload HAProxy).

3. **Test Logrotate Configuration:**

   To ensure the configuration is correct, run:

   ```bash
   sudo logrotate -f /etc/logrotate.d/haproxy
   ```

### Step 4: Repeat for the Second Load Balancer
If you have a second load balancer using HAProxy, repeat these steps on the second server. Ensure that:

- The logging configuration (`/etc/haproxy/haproxy.cfg`) is the same.
- `rsyslog` is set up similarly on the second load balancer.

### Step 5: Verification
After setting up logging and rotation on both load balancers, monitor logs to ensure they are being generated and rotated as expected.

```bash
ls -lh /var/log/haproxy.log*
```

You should see a primary log (`haproxy.log`) and, if rotation has occurred, compressed logs with the format `haproxy.log.1.gz`. If everything works as expected, your logs will be managed and purged automatically after one day.



###Alternative Solution

If you are below error , one can try below alternative solution 

error: skipping "/var/log/haproxy.log" because parent directory has insecure permissions (It's world writable or writable by group which is not "root") Set "su" directive in config file to tell logrotate which user/group should be used for rotation.


The error message indicates that the permissions of the directory `/var/log/` are not secure, which prevents `logrotate` from managing the HAProxy logs correctly. To resolve this issue, you can update the permissions of the directory and add the `su` directive to your logrotate configuration. Here’s how to handle this:

### Solution 1: Fix Directory Permissions
1. **Check Directory Permissions:**

   Verify the current permissions of the `/var/log/` directory:

   ```bash
   ls -ld /var/log/
   ```

   The permissions should look like this: `drwxr-xr-x` or `755`. If they are more permissive, such as `drwxrwxrwx` (777), it could cause the issue.

2. **Modify Directory Permissions:**

   Change the permissions to `755` to ensure that only the root user has write access:

   ```bash
   sudo chmod 755 /var/log/
   ```

3. **Check Permissions for the Log File:**

   Ensure that the `haproxy.log` file itself has secure permissions:

   ```bash
   sudo chown root:adm /var/log/haproxy.log
   sudo chmod 640 /var/log/haproxy.log
   ```

   This will set the correct ownership (`root` user and `adm` group) and restrict permissions to allow read/write access only to the owner and group members.

### Solution 2: Use the `su` Directive in Logrotate
If changing directory permissions is not an option or does not resolve the issue, add the `su` directive to your `logrotate` configuration for HAProxy:

1. **Edit Logrotate Configuration for HAProxy:**

   Open the `logrotate` configuration file for HAProxy:

   ```bash
   sudo nano /etc/logrotate.d/haproxy
   ```

2. **Add the `su` Directive:**

   Modify the configuration as follows to specify the correct user and group for `logrotate`:

   ```bash
   /var/log/haproxy.log {
       daily
       missingok
       rotate 1
       compress
       delaycompress
       notifempty
       create 640 root adm
       su root adm
       postrotate
           /usr/bin/systemctl reload haproxy > /dev/null 2>/dev/null || true
       endscript
   }
   ```

   The `su root adm` directive ensures that `logrotate` uses the correct permissions when handling HAProxy logs. It specifies that `logrotate` should switch to the `root` user and `adm` group during rotation.

### Solution 3: Check Parent Directory Permissions
If you encounter the issue again, check for other parent directories of `/var/log/haproxy.log` to ensure that none are world-writable or have inappropriate permissions:

```bash
ls -ld / /var /var/log
```

Each directory should have permissions similar to:

- `/` should be `drwxr-xr-x` (755)
- `/var` should be `drwxr-xr-x` (755)
- `/var/log` should be `drwxr-xr-x` (755)

### Final Step: Verify Logrotate
After applying the changes, manually test `logrotate` again:

```bash
sudo logrotate -f /etc/logrotate.d/haproxy
```

If there are no errors, the configuration should be functioning correctly, and logs will be rotated daily with logs older than 1 day purged automatically. Let me know if this resolves your issue or if further troubleshooting is needed!

