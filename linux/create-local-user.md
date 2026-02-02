# Create a Local User on Linux

## Purpose
The purpose of creating a local user on Linux is to:
- Provide individual authentication and controlled access to system resources, following the principle of least privilege.
- Enable multi-user environments by isolating user activities, personal files, and configurations in a dedicated account stored in local system files.
- Support standalone servers or services that require host-specific accounts without relying on centralized authentication systems such as LDAP.

---

## Scope
This procedure covers:
- Creating a non-root local user account on a single Linux host using `useradd`.
- Managing user information stored in local files (`/etc/passwd`, `/etc/shadow`, `/etc/group`).
- Standard Linux distributions using local authentication only.

This procedure does **not** cover centralized authentication mechanisms (LDAP, Active Directory).

---

## Prerequisites
- Root access or a user with `sudo` privileges.
- Access to a terminal or command-line interface.
- A unique username that is not already in use.

---

## Procedure
1. Ensure you have administrative privileges.
   ```bash
   sudo -i
   ```
2. Verify that the username does not already exist:
   ```bash
   id newuser
   ```
   If the user does not exist, the command will return:
   ```yaml
   id: ‘newuser’: no such user
   ```
3. Create the user account with a home directory:
   ```bash
   sudo useradd -m newuser
   ```
4. Set a password for the user:
   ```bash
   sudo passwd newuser
   ```
   Enter and confirm the password when prompted.
5. (Optional) Add the user to a secondary group (e.g., sudo):
   ```bash
   sudo usermod -aG sudo newuser
   ```
   
---

## Verification
1. Confirm that the user exists:
   ```bash
   id newuser
   ```
2. Verify the home directory was created:
   ```bash
   ls -ld /home/newuser
   ```
3. Confirm group membership:
   ```bash
   groups newuser
   ```
   
---

## Rollback
1. Remove the user account and home directory:
   ```bash
   sudo userdel -r newuser
   ```
2. Verify that the user has been removed:
   ```bash
   id newuser
   ```
3. Expected output:
   ```yaml
   id: ‘newuser’: no such user
   ```
   
---

## Security Notes
### Risk: Weak Password on New Account
- **Impact :** Brute-force attacks may compromise the account, exposing user files and enabling lateral movement.
- **Mitigation :** Enforce minimum password length and complexity using system password policies (e.g., `pwquality.conf`).
- **Verification :**
```bash
sudo passwd -S newuser
```

### Risk: Unnecessary Sudo Group Membership
- **Impact :** Privilege escalation to root, leading to full system compromise.
- **Mitigation :** Add users to the sudo group only when there is a documented administrative requirement.
- **Verification :**
```bash
groups newuser
sudo -l -U newuser
```

### Risk: Insecure Home Directory Permissions
- **Impact :** Other local users may access private files, violating data isolation.
- **Mitigation :** Restrict home directory permissions to the account owner.
- **Verification :** 
```bash
ls -ld /home/newuser
```

### Risk: No Password Expiration Policy
- **Impact :** Long-lived credentials increase the window of compromise if passwords are leaked.
- **Mitigation :** Enforce password rotation using account aging policies.
- **Verification :** 
```bash
sudo chage -l newuser
```

---

## Troubleshooting
### Symptom
User account was created, but `id newuser` returns *no such user*.

### Possible Causes
- The user creation command failed due to invalid options.
- The username already exists under a different UID or naming conflict.
- Insufficient disk space prevented home directory creation.

### Resolution
```bash
id newuser
sudo useradd -m newuser
df -h /home
journalctl -xe | grep useradd
```

### Symptom
su - newuser fails with authentication failure.

### Possible Causes
- Password was not set after user creation.
- The account is locked.
- The login shell is invalid or missing.

### Resolution
```bash
sudo passwd newuser
sudo passwd -S newuser
grep newuser /etc/passwd
tail -n 20 /var/log/auth.log
```

---

## Audit & Logging
### Relevant Files and Logs
- /var/log/auth.log (Debian/Ubuntu)
- /var/log/secure (RHEL/CentOS)
- /etc/passwd
- /etc/shadow

### Why They Matter
- Authentication logs record account creation attempts, login failures, and privilege escalation events.
- /etc/passwd confirms user existence, UID, home directory, and shell.
- /etc/shadow verifies password hashes, lock status, and expiration policies.

### Example Commands
```bash
grep newuser /etc/passwd
sudo passwd -S newuser
journalctl | grep useradd
ls -l /etc/passwd /etc/shadow
```
