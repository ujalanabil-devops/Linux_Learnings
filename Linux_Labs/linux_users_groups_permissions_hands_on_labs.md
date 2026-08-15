# Linux Users, Groups & Permissions — Complete Hands-On Labs

> **Level:** Beginner → Junior DevOps  
> **Environment:** Ubuntu VM/lab  
> **Important:** Do not run the destructive exercises on production servers.

---

## Table of Contents

1. [Lab Preparation](#0-lab-preparation)
2. [Users](#lab-1--users)
3. [/etc/passwd](#lab-2--etcpasswd)
4. [/etc/shadow](#lab-3--etcshadow)
5. [Groups](#lab-4--groups)
6. [su](#lab-5--su)
7. [sudo](#lab-6--sudo)
8. [User Management](#lab-7--useradd-usermod-userdel)
9. [chmod Basics](#lab-8--chmod-basics)
10. [Numeric Permissions](#lab-9--numeric-permission-challenge)
11. [Symbolic Permissions](#lab-10--symbolic-permissions)
12. [chown](#lab-11--chown)
13. [chgrp](#lab-12--chgrp)
14. [Directory Permissions](#lab-13--directory-permissions)
15. [Production Permission Troubleshooting](#lab-14--production-permission-troubleshooting)
16. [SUID](#lab-15--suid)
17. [SGID](#lab-16--sgid)
18. [Sticky Bit](#lab-17--sticky-bit)
19. [ACL Basics](#lab-18--acl-basics)
20. [ACL Mask](#lab-19--acl-mask)
21. [Production Shared Application](#lab-20--production-style-shared-application)
22. [Monitoring ACL](#lab-21--monitoring-acl)
23. [Break Everything and Troubleshoot](#lab-22--break-everything-and-troubleshoot)
24. [Final Interview Challenge](#lab-23--final-interview-challenge)
25. [Master Command Practice](#master-command-practice)
26. [Final Challenge](#principal-devops-final-challenge)

---

# 0. Lab Preparation

We'll build a simulated production environment:

```text
/opt/devops-lab/
├── app/
│   ├── bin/
│   ├── config/
│   └── logs/
├── shared/
└── secrets/
```

Users:

```text
alice
bob
appuser
monitoring
```

Groups:

```text
devops
developers
appgroup
```

Start a root shell for the lab:

```bash
sudo -i
```

Verify:

```bash
whoami
id
hostname
```

> **Production note:** In real production environments, avoid long-lived root shells. Prefer narrowly scoped `sudo` commands.

---

# Lab 1 — Users

## Objective

Learn users, UIDs, home directories, shells, and primary groups.

### Create users

```bash
useradd -m alice
useradd -m bob
useradd -m appuser
useradd -m monitoring
where,
-m == creates a new user AND creates their home directory
* Without -m, the user is created but no home directory is created, which causes problems for SSH, scripts, and login shells.

Debugging Scenario
* A user cannot SSH into a server even though their password is correct.
Root cause:
Their home directory does not exist → SSH cannot create .ssh/authorized_keys.
Fix:
sudo mkdir /home/username
sudo chown username:username /home/username
or
sudo userdel username
sudo useradd -m username

```

Verify:

```bash
id alice
id bob
id appuser
id monitoring
```

Check home directories:

```bash
ls -ld /home/alice
ls -ld /home/bob
ls -ld /home/appuser
ls -ld /home/monitoring
```

### Set passwords

```bash
passwd alice
passwd bob
passwd appuser
passwd monitoring
```

### Test user switching

```bash
su - alice
```

Run:

```bash
whoami
pwd
id
```

Return:

```bash
exit
```

## Challenge

Find:

- Alice's UID
- Alice's primary GID
- Alice's home directory
- Alice's shell

Useful commands:

```bash
id alice
getent passwd alice

* getent = “get entries” from system databases (passwd, group, hosts, etc.).
* getent returns users from all identity sources(local Linux user, LDAP user, Active Directory user, NIS user) not just local accounts.
```

---

# Lab 2 — `/etc/passwd`

## Objective

Understand where Linux stores basic user-account information.

View:

```bash
cat /etc/passwd
```

Find Alice:

```bash
grep '^alice:' /etc/passwd
^ = beginning of line
alice: = username followed by a colon (the passwd format)
```

Preferred lookup:

```bash
getent passwd alice
```

Example:

```text
alice:x:1001:1001::/home/alice:/bin/bash
```

Fields:

```text
username
password placeholder
UID
primary GID
GECOS/comment
home directory
login shell
```

## Exercises

```bash
getent passwd alice
getent passwd bob
getent passwd appuser
getent passwd monitoring
```

Find UIDs:

```bash
id -u alice
id -u bob
id -u appuser
id -u monitoring
```

## Challenge

Without using `id`, find Alice's UID from:

```bash
getent passwd alice
```

## Production note

Prefer `getent` when possible because it respects the system's configured identity sources, which may include LDAP/AD/NSS rather than only local files.

---

# Lab 3 — `/etc/shadow`

## Objective

Understand password hashes and account/password aging.

Check permissions:

```bash
ls -l /etc/shadow
```

Try:

```bash
cat /etc/shadow
```

As root:

```bash
cat /etc/shadow
```

Find Alice:

```bash
grep '^alice:' /etc/shadow
```

Check password status:

```bash
passwd -S alice

output:-
alice P 08/13/2026 0 99999 7
where,
P → password is set
password last changed on 08/13/2026
password aging rules follow (min/max days, warning days)

What does the “L” or “P” in passwd -S username mean?
Expected answer:
P → password is set
L → account is locked
NP → no password set
LK → locked via usermod -L or passwd -l

Confusing password expiration with account locking.
expired password → user must change password
locked account → user cannot log in at all
```

Check password aging:

```bash
chage -l alice
```

## Exercise — Lock,Unlock,change password,

Change Password:
```
sudo passwd alice
```
Set Password Aging (expiry rules):
```
View current aging: sudo chage -l alice
Set password to expire every 90 days: sudo chage -M 90 alice
Force user to change password at next login: sudo chage -d 0 alice
Set minimum days between password changes (e.g., 1 day): sudo chage -m 1 alice
Set warning period before expiry (e.g., 7 days): sudo chage -W 7 alice

Note: Setting -M 0 → password expires immediately → user locked out.
```

Password Expire vs Account Expire:
If the password expires: user can still log in, but they are forced to change their password immediately,SSH may deny login if password change on login .
If the account expires: user cannot log in at all, even with the correct password, even with SSH keys, even if password is valid.
```
sudo chage -E 2026-12-31 alice
active expire account: sudo chage -E -1 alice (-1 means “never expire”)
```
Remove a Password (make account passwordless):
```
sudo passwd -d alice

Real‑world use case: Service accounts that should never have a password.
```
Password inactive:
First → password expires
Then → user has X “inactive days”
After X days → account becomes fully locked

example:
Password expires on Aug 10
Password inactive = 7 days
Aug 10: Password expired → user must change password
Aug 11–17: Password inactive period → user can still log in only to change password
Aug 18: Account locked → user cannot log in at all
```
set password inactive days: sudo chage -I 7 alice
remove password inactive period: sudo chage -I -1 alice
```

Disable a Password (lock the account):

```bash
passwd -l alice

Adds ! in front of the password hash in /etc/shadow.
User cannot log in, even with the correct password.
```
Check:

```bash
passwd -S alice
```

Unlock:

```bash
passwd -u alice
```

Verify:

```bash
passwd -S alice
```

## Production note

Do not manually edit `/etc/shadow` with a normal editor. Use `passwd`, `usermod`, and `chage`.

---

# Lab 4 — Groups

## Objective

Understand primary and supplementary groups.

Create:

```bash
groupadd devops
groupadd developers
groupadd appgroup
```

Check:

```bash
getent group devops
getent group developers
getent group appgroup
```

Add users:

```bash
usermod -aG devops alice
usermod -aG devops bob

usermod -aG developers alice
usermod -aG developers bob

usermod -aG appgroup appuser

where,
-a → append (add without removing existing groups)
-G devops → add user to the devops supplementary group
alice → the username
```

Verify:

```bash
id alice
id bob
id appuser
```

Check groups:

```bash
getent group devops
getent group developers
getent group appgroup
```

## Important Experiment

Run:

```bash
usermod -G devops alice
```

Then:

```bash
id alice
```

Observe what happened to Alice's supplementary groups.

Restore them:

```bash
usermod -aG devops alice
usermod -aG developers alice
```

## Key Lesson

Prefer:

```bash
usermod -aG group user
```

because `-a` means append.

Be careful with:

```bash
usermod -G group user
```

because it replaces the supplementary group list.

---

# Lab 5 — `su`

## Objective

Practice switching user accounts.

Switch to Alice:

```bash
su - alice
```

Check:

```bash
whoami
id
pwd
echo "$HOME"
echo "$SHELL"
```

Return:

```bash
exit
```

Compare:

```bash
su appuser

Switches to the user appuser
Keeps your current environment
Does NOT load:
.bashrc
.profile
.bash_profile
user’s PATH
user’s environment variables
```

with:

```bash
su - appuser
Simulates a real login as appuser

Loads:
/etc/profile
~appuser/.bash_profile
~appuser/.profile
~appuser/.bashrc
Sets correct:PATH,HOME,SHELL,environment variables
```
# Lab 6 — `sudo`

## Objective

Understand controlled privilege escalation.

Check your sudo permissions:

```bash
sudo -l
it shows:-
* which commands you can run with sudo.
* whether you need a password.
* which hosts the rules apply to
* whether you have NOPASSWD privileges.
* whether you have full root access
```

Test:

```bash
sudo whoami

"who you become when running a command with sudo."
```

Expected:

```text
root
```

Test:

```bash
sudo id
```

Run a command as another user:

```bash
“Run the command whoami as appuser, using sudo.”

sudo -u appuser whoami

* runs only that command as appuser
* does not switch your shell
* does not ask for appuser’s password
* requires your sudo permissions, not appuser’s

* used in production: to ensure: permissions are correct,environment variables are correct,the app behaves the same way it does under systemd.

```

Expected:

```text
appuser
```
## Production note

Prefer narrowly scoped `sudo` commands over permanently working as root.

---

# Lab 7 — `useradd`, `usermod`, `userdel`

## Create a temporary user

```bash
useradd -m testuser
```

Change shell:

```bash
usermod -s /bin/bash testuser

Changing the shell is necessary when:
* A user cannot log in because their shell is set to /usr/sbin/nologin or /bin/false
* You want a service account to run interactive commands
* You want to enable scripting for a user
* You need to fix a misconfigured shell after a migration

```

Add to group:

```bash
usermod -aG devops testuser
```

Verify:

```bash
id testuser
getent passwd testuser
```

Lock:

```bash
usermod -L testuser
```

Unlock:

```bash
usermod -U testuser
```

Delete without home directory:

```bash
userdel testuser

Access the directory as root: ls -la /home/testuser
change ownership: sudo chown -R $USER:$USER /home/testuser
Copy files: sudo cp -r /home/testuser /backup/
Move files: sudo mv /home/testuser /home/newuser/
Archive: sudo tar czf testuser_backup.tar.gz /home/testuser
```

Check:

```bash
getent passwd testuser
ls -ld /home/testuser
```

Create another:

```bash
useradd -m testuser2
```

Delete with the home directory:

```bash
userdel -r testuser2
```

## Production Warning

Never blindly run:

```bash
userdel -r
```

on a production server.

Before deleting a service account, investigate:

```bash
ps aux
```

and:

```bash
find /opt -user username
```

---

# Lab 8 — `chmod` Basics

Create:

```bash
mkdir -p /opt/devops-lab
cd /opt/devops-lab

touch file644
touch file755
touch file600
touch file750
```

Set permissions:

```bash
chmod 644 file644
chmod 755 file755
chmod 600 file600
chmod 750 file750
```

Check:

```bash
ls -l
```

Understand:

```text
644 = rw-r--r--
755 = rwxr-xr-x
600 = rw-------
750 = rwxr-x---
```

Permission values:

```text
r = 4
w = 2
x = 1
```

Therefore:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
3 = -wx
2 = -w-
1 = --x
0 = ---
```

---

# Lab 9 — Numeric Permission Challenge

Create:

```bash
touch test1
touch test2
touch test3
touch test4
touch test5
```

Set:

```text
test1 → 700
test2 → 640
test3 → 660
test4 → 740
test5 → 755
```

Use:

```bash
chmod 700 test1
chmod 640 test2
chmod 660 test3
chmod 740 test4
chmod 755 test5
```

Verify:

```bash
ls -l test*
```

---

# Lab 10 — Symbolic Permissions

Create:

```bash
touch symbolic.txt
chmod 644 symbolic.txt
```

Check:

```bash
ls -l symbolic.txt
```

Add execute for owner:

```bash
chmod u+x symbolic.txt
```

Remove write from owner:

```bash
chmod u-w symbolic.txt
```

Add write to group:

```bash
chmod g+w symbolic.txt
```

Remove read from others:

```bash
chmod o-r symbolic.txt
```

Set exact permissions symbolically:

```bash
chmod u=rw,g=r,o= symbolic.txt
```

Check:

```bash
ls -l symbolic.txt
```

## Important notation

```text
u = owner
g = group
o = others
a = all
```

---

# Lab 11 — `chown`

Create:

```bash
mkdir -p /opt/devops-lab/app
touch /opt/devops-lab/app/application.conf
```

Check:

```bash
ls -l /opt/devops-lab/app/application.conf
```

Change owner:

```bash
chown appuser /opt/devops-lab/app/application.conf
```

Change owner and group:

```bash
chown appuser:appgroup /opt/devops-lab/app/application.conf
```

Verify:

```bash
ls -l /opt/devops-lab/app/application.conf
```

## Production Warning

Be extremely careful with:

```bash
chown -R
```

A recursive ownership change can break an application.

---

# Lab 12 — `chgrp`

Create:

```bash
touch /opt/devops-lab/app/application.log
```

Set owner:

```bash
chown appuser /opt/devops-lab/app/application.log
```

Change group:

```bash
chgrp appgroup /opt/devops-lab/app/application.log
```

Verify:

```bash
ls -l /opt/devops-lab/app/application.log
```

Equivalent:

```bash
chown appuser:appgroup /opt/devops-lab/app/application.log
```

---

# Lab 13 — Directory Permissions

## Objective

Understand the special meaning of permissions on directories.

Create:

```bash
mkdir /opt/devops-lab/shared
chown appuser:devops /opt/devops-lab/shared
chmod 750 /opt/devops-lab/shared
```

Check:

```bash
ls -ld /opt/devops-lab/shared
```

Create a file as appuser:

```bash
sudo -u appuser touch /opt/devops-lab/shared/app.txt

if you get permission denied:
even though you are ussing sudo -u appuser ,the appuser account needs execute(x) permission on every parent directory.
check permission : namei -l /opt/devops-lab/shared/app.txt
```

Check:

```bash
ls -l /opt/devops-lab/shared
```

## Remember

For directories:

```text
r = list contents
w = create/delete directory entries
x = enter/traverse directory
```

## Challenge

Experiment with directories having:

```text
700
750
755
770
```

Observe what different users can do.

---

# Lab 14 — Production Permission Troubleshooting

## Scenario

An application runs as:

```text
appuser
```

It reports:

```text
Permission denied
```

Create:

```bash
mkdir -p /opt/devops-lab/app/config
touch /opt/devops-lab/app/config/application.yml
```

Deliberately create the problem:

```bash
chown root:root /opt/devops-lab/app/config/application.yml
chmod 600 /opt/devops-lab/app/config/application.yml
```

Test:

```bash
sudo -u appuser cat /opt/devops-lab/app/config/application.yml
```

Expected:

```text
Permission denied
```

## Troubleshooting

Do NOT use:

```bash
chmod 777
```

Investigate:

```bash
ls -l /opt/devops-lab/app/config/application.yml
id appuser
namei -l /opt/devops-lab/app/config/application.yml
```

Fix:

```bash
chown appuser:appgroup /opt/devops-lab/app/config/application.yml
chmod 640 /opt/devops-lab/app/config/application.yml
```

Test:

```bash
sudo -u appuser cat /opt/devops-lab/app/config/application.yml
```

## Production Mindset

Always identify:

1. Which user?
2. Which group?
3. Which file?
4. Which permissions?
5. Can the user traverse the path?
6. Are ACLs involved?
7. Is SELinux/AppArmor involved?

---

# Lab 15 — SUID
- SUID (Set‑User‑ID) is a special Linux permission bit that makes a program run with the file owner’s privileges, not the privileges of the user who launched it.
## Objective

Understand SUID.
When a file has the SUID bit set:
Any user who runs that program
Temporarily becomes the file owner (usually root)
Only for the duration of that program

Why SUID exists:
Some tasks require elevated privileges, but you don’t want to give users full root access.
Examples:
passwd → needs root to modify /etc/shadow
ping → needs raw socket access
sudo → needs root to escalate privileges
SUID should never be set on scripts
SUID should be avoided unless absolutely necessary

```bash
find /usr/bin -perm -4000 -type f 2>/dev/null

1. /usr/bin: Search only inside the /usr/bin directory.
2. -perm -4000: Match files with the SUID bit set.4 = SUID,000 = other permission bits,-4000 = “file has SUID bit ON”
3. -type f: Only show regular files, not directories.
4. 2>/dev/null: Hide error messages (like “permission denied”) by redirecting stderr to /dev/null.
```

Inspect:

Typical SUID binaries include:
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/newgrp
These are programs that must run with elevated privileges (usually root) to modify system files.
If you see more than these, you should investigate.
SUID binaries are one of the top vectors for privilege escalation.
If you see a weird SUID binary like:→ That’s a security red flag.

```bash
ls -l /usr/bin/passwd

| Binary | Normal? | Notes |
| --- | --- | --- |
| unmount | ✔ | Expected on some distros |
| su | ✔ | Required for user switching |
| sudo.ws | ❓ | **Unusual — investigate** |
| fusermount3 | ✔ | Required for FUSE |
| pkexec | ✔ | Normal but historically vulnerable |
| mount | ✔ | Required for mounting |
| ntfs-3g | ✔ | Required for NTFS mounts |
```

Look for:

```text
-rwsr-xr-x
```

The `s` indicates SUID.

Find SUID files across the system:

```bash
find / -perm -4000 -type f 2>/dev/null
```

## Challenge

Choose three SUID binaries and answer:

1. Who owns them?
2. Why might SUID be required?
3. What is the security risk of a vulnerable SUID program?

## Production Warning

Do not randomly remove SUID from system binaries.

---

# Lab 16 — SGID

## Objective

Use SGID for shared directories.

Create:

```bash
mkdir /opt/devops-lab/team
chgrp devops /opt/devops-lab/team
chmod 2775 /opt/devops-lab/team
```

Check:

```bash
ls -ld /opt/devops-lab/team
```

Expected pattern:

```text
drwxrwsr-x
```

Create as Alice:

```bash
sudo -u alice touch /opt/devops-lab/team/alice.txt
```

Create as Bob:

```bash
sudo -u bob touch /opt/devops-lab/team/bob.txt
```

Check:

```bash
ls -l /opt/devops-lab/team
```

Both files should inherit the directory's group.

## Key Concept

For a directory:

```text
SGID = newly created files/directories inherit the directory's group
```

---

# Lab 17 — Sticky Bit

## Objective

Protect files in a shared writable directory.

Create:

```bash
mkdir /opt/devops-lab/dropbox
chmod 1777 /opt/devops-lab/dropbox
```

Check:

```bash
ls -ld /opt/devops-lab/dropbox
```

Expected:

```text
drwxrwxrwt
```

Create Alice's file:

```bash
sudo -u alice touch /opt/devops-lab/dropbox/alice.txt
```

Try deleting as Bob:

```bash
sudo -u bob rm /opt/devops-lab/dropbox/alice.txt
```

Normally this should fail.

## Key Concept

```text
Sticky bit = users can share a writable directory without normally being able to delete each other's files.
```

Common example:

```bash
ls -ld /tmp
```

---

# Lab 18 — ACL Basics

## Objective

Give a specific user access without changing the main owner/group model.

Create:

```bash
touch /opt/devops-lab/app/logs.log
chown appuser:appgroup /opt/devops-lab/app/logs.log
chmod 640 /opt/devops-lab/app/logs.log
```

Test monitoring:

```bash
sudo -u monitoring cat /opt/devops-lab/app/logs.log
```

It should fail.

Give monitoring read access:

```bash
setfacl -m u:monitoring:r /opt/devops-lab/app/logs.log
```

Inspect:

```bash
getfacl /opt/devops-lab/app/logs.log
```

Test:

```bash
sudo -u monitoring cat /opt/devops-lab/app/logs.log
```

Remove:

```bash
setfacl -x u:monitoring /opt/devops-lab/app/logs.log
```

Verify:

```bash
getfacl /opt/devops-lab/app/logs.log
```

## Important

If `ls -l` shows:

```text
-rw-r-----+
```

the `+` means an extended ACL exists.

---

# Lab 19 — ACL Mask

Create:

```bash
touch /opt/devops-lab/mask-test
```

Give Alice full ACL permissions:

```bash
setfacl -m u:alice:rwx /opt/devops-lab/mask-test
```

Inspect:

```bash
getfacl /opt/devops-lab/mask-test
```

Now restrict the mask:

```bash
setfacl -m m:r /opt/devops-lab/mask-test
```

Check:

```bash
getfacl /opt/devops-lab/mask-test
```

You may see:

```text
user:alice:rwx
mask::r--
```

The mask limits the effective permissions.

Restore:

```bash
setfacl -m m:rwx /opt/devops-lab/mask-test
```

---

# Lab 20 — Production-Style Shared Application

## Scenario

Build:

```text
/opt/devops-lab/production
├── bin/
├── config/
│   └── application.yml
└── logs/
    └── application.log
```

Create:

```bash
mkdir -p /opt/devops-lab/production/{bin,config,logs}
touch /opt/devops-lab/production/config/application.yml
touch /opt/devops-lab/production/logs/application.log
```

Set initial ownership:

```bash
chown -R appuser:appgroup /opt/devops-lab/production
```

Set directory permissions:

```bash
chmod 750 /opt/devops-lab/production
chmod 750 /opt/devops-lab/production/bin
chmod 750 /opt/devops-lab/production/config
chmod 750 /opt/devops-lab/production/logs
```

Set file permissions:

```bash
chmod 640 /opt/devops-lab/production/config/application.yml
chmod 640 /opt/devops-lab/production/logs/application.log
```

Now configure group access so `devops` can read and traverse the application.

Use:

```bash
chgrp -R devops /opt/devops-lab/production
```

Then verify:

```bash
ls -lR /opt/devops-lab/production
```

## Requirements

```text
appuser:
    full access where required

devops:
    read files
    traverse directories

others:
    no access
```

Do not solve the problem with `777`.

---

# Lab 21 — Monitoring ACL

Monitoring needs to read logs but should not read application configuration.

Give monitoring read access to the log:

```bash
setfacl -m u:monitoring:r /opt/devops-lab/production/logs/application.log
```

Test:

```bash
sudo -u monitoring cat /opt/devops-lab/production/logs/application.log
```

Try configuration:

```bash
sudo -u monitoring cat /opt/devops-lab/production/config/application.yml
```

The configuration should remain protected.

## Production Principle

Give each account the minimum permissions required to perform its job.

This is the **principle of least privilege**.

---

# Lab 22 — Break Everything and Troubleshoot

This is the most important lab.

## Problem 1 — Wrong Owner

Break:

```bash
chown root:root /opt/devops-lab/production/config/application.yml
```

Test:

```bash
sudo -u appuser cat /opt/devops-lab/production/config/application.yml
```

Troubleshoot:

```bash
ls -l /opt/devops-lab/production/config/application.yml
id appuser
namei -l /opt/devops-lab/production/config/application.yml
```

Fix using the minimum required permissions.

---

## Problem 2 — Wrong Directory Permissions

Break:

```bash
chmod 600 /opt/devops-lab/production/logs
```

Test:

```bash
sudo -u monitoring cat /opt/devops-lab/production/logs/application.log
```

Troubleshoot:

```bash
ls -ld /opt/devops-lab/production/logs
namei -l /opt/devops-lab/production/logs/application.log
```

Fix it.

---

## Problem 3 — Wrong Group

Break:

```bash
chgrp root /opt/devops-lab/production
```

Test:

```bash
sudo -u alice ls /opt/devops-lab/production
```

Investigate:

```bash
ls -ld /opt/devops-lab/production
id alice
```

Fix it.

---

## Problem 4 — Unexpected ACL

Add:

```bash
setfacl -m u:alice:rwx /opt/devops-lab/production/logs/application.log
```

Inspect:

```bash
getfacl /opt/devops-lab/production/logs/application.log
```

Identify the unexpected permission.

Remove:

```bash
setfacl -x u:alice /opt/devops-lab/production/logs/application.log
```

Verify:

```bash
getfacl /opt/devops-lab/production/logs/application.log
```

---

# Lab 23 — Final Interview Challenge

## Scenario

An application runs as:

```text
appuser
```

The application reports:

```text
Permission denied
```

Target:

```text
/opt/webapp/config/application.yml
```

You have no information about what is wrong.

## Rule

Do not change anything until you investigate.

Start:

```bash
whoami
```

Then:

```bash
id appuser
```

Then:

```bash
ls -l /opt/webapp/config/application.yml
```

Then:

```bash
namei -l /opt/webapp/config/application.yml
```

Then:

```bash
getfacl /opt/webapp/config/application.yml
```

## Answer these questions

```text
1. Who owns the file?
2. What is the group?
3. What are the owner permissions?
4. What are the group permissions?
5. What are the other permissions?
6. Is appuser in the relevant group?
7. Can appuser traverse every directory?
8. Is an ACL present?
9. Is there an ACL mask?
10. What is the smallest safe fix?
```

## Production Rule

Never use this as your first response:

```bash
chmod 777
```

---

# Master Command Practice

By the end of these labs, you should be comfortable with:

## User Management

```bash
useradd
usermod
userdel
passwd
chage
```

## Group Management

```bash
groupadd
groupdel
gpasswd
groups
id
getent
```

## Privilege

```bash
su
sudo
sudo -l
```

## Permissions

```bash
chmod
chown
chgrp
ls -l
stat
```

## Special Permissions

```text
SUID
SGID
Sticky Bit
```

## ACL

```bash
setfacl
getfacl
```

## Troubleshooting

```bash
namei -l
find
id
ls -l
ps
```

---

# Principal DevOps Final Challenge

After completing all labs:

1. Delete the lab environment.
2. Recreate the users and groups from memory.
3. Recreate the application directory structure.
4. Configure ownership.
5. Configure permissions.
6. Configure SGID where appropriate.
7. Configure an ACL for monitoring.
8. Verify access as each user.
9. Deliberately break five permissions.
10. Troubleshoot each without using `chmod 777`.

Your troubleshooting thought process should become:

```text
Permission Denied
       |
       v
Who am I?
       |
       v
id
       |
       v
Who owns the file?
       |
       v
ls -l
       |
       v
What permissions exist?
       |
       v
chmod
       |
       v
Can I traverse the path?
       |
       v
namei -l
       |
       v
Are ACLs involved?
       |
       v
getfacl
       |
       v
Is SELinux/AppArmor involved?
       |
       v
Apply the minimum required fix
```

---

# Next Topics

After mastering these labs, move to:

```text
1. Processes
   ps, top, htop, pgrep, pkill, kill

2. systemd
   systemctl, services, journalctl

3. Linux Networking
   ip, ss, ping, curl, dig, traceroute

4. Storage
   df, du, lsblk, mount

5. Logs
   journalctl, /var/log

6. Bash Scripting

7. Python Automation

8. Docker

9. Kubernetes

10. Prometheus + Grafana
```

The key DevOps skill is not memorizing `chmod 755`. It is being able to answer:

> **"Which identity needs which access to which resource, and what is the minimum permission required?"**
