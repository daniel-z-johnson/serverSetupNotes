# Debian Server Initial Setup

Set up automatic updates and a non-root administrator account, then configure SSH key login.

## Before you start

- This guide assumes Debian with systemd and an SSH server already installed. It has not yet been tested on a live server; record the Debian version here after testing.
- Replace `username` with your chosen account name and `server` with the server's IP address or hostname in every command.
- Start with a root shell on the server. If using an existing administrator account instead, prefix the root commands in steps 1–3 with `sudo`.
- Keep your original server session open until a fresh SSH login and sudo access both work after the final change. Make sure you have access to a provider or recovery console.
- Commands marked **Local computer** require an OpenSSH client, including `ssh-copy-id`.

## 1. Update the server

**Server — root:**

```sh
apt update
apt upgrade
```

Review package prompts. If updates require a reboot, schedule it and reconnect before continuing.

## 2. Enable unattended upgrades

**Server — root:**

```sh
apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

Choose **Yes** to enable automatic updates. Check `/etc/apt/apt.conf.d/20auto-upgrades` for these settings:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Review the enabled origins in `/etc/apt/apt.conf.d/50unattended-upgrades` to confirm which updates may be installed automatically, including security updates for your Debian release. Review the automatic reboot settings there as well.

Enable and inspect the APT timers that schedule the work:

```sh
systemctl enable --now apt-daily.timer apt-daily-upgrade.timer
systemctl list-timers --all apt-daily.timer apt-daily-upgrade.timer
unattended-upgrade --dry-run --debug
```

Confirm both timers appear with a next run scheduled. Review the dry-run output for errors and the expected allowed origins. No eligible updates is a valid result. A running `unattended-upgrades` service alone does not confirm that automatic updates are scheduled correctly.

After a scheduled run, inspect `/var/log/unattended-upgrades/unattended-upgrades.log` for the result.

## 3. Create an administrator account

**Server — root:**

```sh
apt install sudo
adduser username
usermod -aG sudo username
```

Set a strong account password when prompted. It will still be used for sudo after SSH password login is disabled.

## 4. Set up SSH keys

**Local computer:** Use an existing SSH key, or generate an Ed25519 key if you do not already have one:

```sh
ssh-keygen -t ed25519
```

Choose a passphrase to protect the private key. Do not overwrite an existing key. The following commands assume the default key path; substitute your chosen path if different.

Copy the public key to the new server account:

```sh
ssh-copy-id -i ~/.ssh/id_ed25519.pub username@server
```

Verify the server's host key fingerprint through a trusted source, such as your provider's console, before accepting a new host key. This initial key installation may request the new account's password. `ssh-copy-id` installs the public key into the remote account's `authorized_keys`; keep the private key on your local computer.

If the server already disallows password login, use your existing root or administrator session to install the public key into the new user's `~/.ssh/authorized_keys`. Ensure the new user owns the `.ssh` directory and file, with permissions `700` and `600`, respectively, before continuing.

## 5. Verify key login and sudo access

**Local computer — second terminal:** Keep your original server session open and test a fresh connection with password fallback disabled:

```sh
ssh -o ControlPath=none -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no -i ~/.ssh/id_ed25519 username@server
```

No server account password should be requested. Your local private key may still require its passphrase.

**Server — new user's session:**

```sh
sudo whoami
```

Enter the new user's account password if prompted. The output must be `root`. Resolve any login or sudo problems before continuing.

## 6. Disable root and password SSH login

**Server — new user's session:** Back up the configuration before editing. Use a different backup destination if this backup already exists.

```sh
sudo cp -a /etc/ssh /etc/ssh.backup-before-hardening
sudoedit /etc/ssh/sshd_config
```

Set these options in the global section, before any `Match` blocks:

```text
PermitRootLogin no
PermitEmptyPasswords no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
```

Inspect files included from `/etc/ssh/sshd_config.d/` as well. OpenSSH normally uses the first value it reads for each option, and Debian includes these files near the start of the main configuration. Update conflicting settings where they are defined instead of simply appending duplicate lines. Review any `Match` blocks that could change authentication for particular connections.

Validate the configuration and inspect the effective settings:

```sh
sudo /usr/sbin/sshd -t
sudo /usr/sbin/sshd -T | grep -E '^(permitrootlogin|permitemptypasswords|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
```

The syntax check should exit successfully without output. The effective settings should match the values above. If you use `Match` blocks, also check the relevant connection with `sshd -T -C`, supplying its user, host, and address as described in the SSH manual.

Only after validation succeeds, reload Debian's SSH service:

```sh
sudo systemctl reload ssh
sudo systemctl status ssh --no-pager
```

**Local computer — another terminal:** Repeat the fresh key login command from step 5, then run `sudo whoami` on the server again. Close the original session only after both checks succeed.

If validation or login fails, keep the original session open, correct or restore the configuration files you changed, validate again, and reload SSH. Use the recovery console if you cannot connect.

## References

- [Debian periodic updates](https://wiki.debian.org/PeriodicUpdates)
- [Unattended-upgrades documentation](https://sources.debian.org/src/unattended-upgrades/2.12/README.md)
- [SSH configuration and precedence](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html)
- [SSH configuration validation](https://manpages.debian.org/trixie/openssh-server/sshd.8.en.html)
- [Installing SSH public keys](https://manpages.debian.org/trixie/openssh-client/ssh-copy-id.1.en.html)
