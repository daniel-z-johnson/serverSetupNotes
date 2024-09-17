# serverSetupNotes

## Secure Server steps for Debian systems
1. upadate server
    1. `apt update && apt upgrade`\
  
2. Enabled unattended upgrades
   1. `apt install unattended-upgrades`
   2. `dpkg-reconfigure --priority=low unattended-upgrades`
   3. `systemctl start unattended-upgrades`
   4. `systemctl enable unattended-upgrades`
   5. Make sure it is running, should see it as `Active: active (running)`
   6. `systemcrl status unattended-upgrades`
   7. Dry run to make sure everything looks good
   8. `unattended-upgrades --dry-run --debug`
  
3. 
