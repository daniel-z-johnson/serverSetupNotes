# serverSetupNotes

## Secure Server steps for Debian systems
1. Update Server
2. Enable unattended upgrades
3. Add new user to avoid having to log in as root
4. Add user to sudo group
5. Log out and log in as new user and make sure sudo is enabled for user
6. Disabled passwords and use public/private key to log in (rsa)


### Upadate Server
1. `apt update && apt upgrade`
  
### Enabled unattended upgrades
1. `apt install unattended-upgrades`
2. `dpkg-reconfigure --priority=low unattended-upgrades`
3. `systemctl start unattended-upgrades`
4. `systemctl enable unattended-upgrades`
5. Make sure it is running, should see it as `Active: active (running)`
6. `systemcrl status unattended-upgrades`
7. Dry run to make sure everything looks good
8. `unattended-upgrades --dry-run --debug`
  
### Add new user to avoid having to log in as root
1. `adduser username`
  
### Add user to sudo group
1. `usermod -aG sudo username`
  
### Log out and log in as new user and make sure sudo is enabled for user
### Disabled passwords and use public/private key to log in (rsa)
   1. log in as user created above
   2. `mkdir ~/.ssh && chmod 700 ~/.ssh`
   3. `ssh-copi-id <username>@server`
   4. `ssh <username>@server` no password should be required
   5. edit `/etc/ssh/sshd_config`
   6. add
      1. `port <some port above 1024>`
      2. `PermitRootlogin no`
      3. `PermitAuthEmptyPasswords no`
      4. `PasswordAuthentication no`
     
   7. `systemctl restart sshd`
      - `ssh -p newPort username@server`
   9. In seperate terminal ssh into server to make sure everything is working
  
