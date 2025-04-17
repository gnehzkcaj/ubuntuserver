# Ubuntu Update, Upgrade, and Restart Automation Script

Here's a script that automates updates, upgrades, and performs a midnight restart on Ubuntu. I'll provide two versions: one as a direct bash script and another as a systemd service/timer setup for more robust scheduling.

## Option 1: Simple Bash Script with Cron

1. First, create the script file:

```bash
#!/bin/bash

# Update package lists
echo "Starting system update..."
apt-get update -y

# Upgrade all packages
echo "Starting system upgrade..."
apt-get upgrade -y

# Perform distribution upgrade (if available)
echo "Starting distribution upgrade..."
apt-get dist-upgrade -y

# Clean up unused packages
echo "Cleaning up..."
apt-get autoremove -y
apt-get autoclean

# Check if reboot is required
if [ -f /var/run/reboot-required ]; then
    echo "Reboot required. Scheduling reboot at midnight..."
    # Schedule reboot at midnight
    echo "0 0 * * * root /sbin/shutdown -r now" > /etc/cron.d/auto_reboot
    service cron reload
else
    echo "No reboot required. Removing any scheduled reboots..."
    rm -f /etc/cron.d/auto_reboot
    service cron reload
fi

echo "Update process completed."
```

2. Save this as `/usr/local/bin/auto_update.sh` and make it executable:
```bash
sudo chmod +x /usr/local/bin/auto_update.sh
```

3. Add it to root's crontab to run daily (e.g., at 11:30 PM):
```bash
sudo crontab -e
```
Add this line:
```
30 23 * * * /usr/local/bin/auto_update.sh
```

## Option 2: Systemd Service and Timer (More Robust)

1. Create a service file `/etc/systemd/system/auto-update.service`:
```ini
[Unit]
Description=Automated System Update and Upgrade
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/auto_update.sh
```

2. Create a timer file `/etc/systemd/system/auto-update.timer`:
```ini
[Unit]
Description=Run auto-update daily at 11:30 PM

[Timer]
OnCalendar=*-*-* 23:30:00
Persistent=true

[Install]
WantedBy=timers.target
```

3. Create the script `/usr/local/bin/auto_update.sh` (same as above but with reboot modification):
```bash
#!/bin/bash

# Log file
LOG_FILE="/var/log/auto-update.log"

{
    echo "=== Update started at $(date) ==="
    
    # Update package lists
    echo "Starting system update..."
    apt-get update -y

    # Upgrade all packages
    echo "Starting system upgrade..."
    apt-get upgrade -y

    # Perform distribution upgrade (if available)
    echo "Starting distribution upgrade..."
    apt-get dist-upgrade -y

    # Clean up unused packages
    echo "Cleaning up..."
    apt-get autoremove -y
    apt-get autoclean

    # Check if reboot is required
    if [ -f /var/run/reboot-required ]; then
        echo "Reboot required. Scheduling reboot at midnight..."
        # Schedule immediate reboot at midnight
        echo "0 0 * * * root /sbin/shutdown -r now" > /etc/cron.d/auto_reboot
        service cron reload
    else
        echo "No reboot required. Removing any scheduled reboots..."
        rm -f /etc/cron.d/auto_reboot
        service cron reload
    fi

    echo "=== Update completed at $(date) ==="
} >> "$LOG_FILE" 2>&1
```

4. Make the script executable:
```bash
sudo chmod +x /usr/local/bin/auto_update.sh
```

5. Enable and start the timer:
```bash
sudo systemctl daemon-reload
sudo systemctl enable auto-update.timer
sudo systemctl start auto-update.timer
```

## Notes:

1. The script will:
   - Update package lists
   - Upgrade all installed packages
   - Perform a distribution upgrade if available
   - Clean up unused packages
   - Check if a reboot is needed
   - Schedule a midnight reboot if required

2. Logs are written to `/var/log/auto-update.log` in the systemd version

3. To check the status of the systemd timer:
```bash
sudo systemctl list-timers auto-update.timer
```

4. To view logs:
```bash
sudo journalctl -u auto-update.service
# or
tail -f /var/log/auto-update.log
```

Choose the option that best fits your needs. The systemd version is generally more robust for system services.
