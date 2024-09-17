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
  
3. Add new user to avoid having to log in as root
   1. `adduser dzjohnson`
  
4. Add user to sudo group
   1. `usermod -aG sudo dzjohnson`
  
5. Log out and log in as new user and make sure sudo is enabled for user
6. Disabled passwords and use public/private key to log in (rsa, ed25519)
   1. log in as user created above
   2. `mkdir ~/.ssh && chmod 700 ~/.ssh`
   3. `ssh-copi-id <username>@server`
