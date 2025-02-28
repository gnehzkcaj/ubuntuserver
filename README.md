# ubuntu server

### View Last Login Information
To see the last login times for all users, use the last command:
```
last
```

To check the last login for a specific user, use:
```
last <username>
```

Check SSH Login Activity
If you're using SSH, you can check the SSH logs for login activity:
```
grep "sshd" /var/log/auth.log
```

Check User Login History
To see a user's login history, you can use the lastlog command:
```
lastlog
```

Install Fail2Ban:
```
sudo apt update
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
maxretry = 3
bantime = 1h

sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

```
