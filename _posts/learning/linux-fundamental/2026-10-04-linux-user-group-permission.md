---
title: "Who Are You?, I Cannot Let You In"
date: 2026-10-04 20:00:00 +0700
categories: [Guide]
tags: [Linux]
image: /assets/img/posts/linux-user-group-permission/capture0.jpeg
alt: "Tux with a padlock and a group of users"
description: "Figure out how Linux decides who gets in and what they can do once inside: users, groups, sudo, and rwx permissions."
learning: true
course: "Linux Fundamental"
module: 2
pin: false
quizzes:
  pretest:
    title: "Pre-test"
    questions:
      - q: "Which file holds the list of user accounts on a Linux system (name, UID, home directory, shell)?"
        options:
          - "/etc/group"
          - "/etc/passwd"
          - "/etc/hosts"
          - "/etc/fstab"
        answer: 2
        explanation: "`/etc/passwd` has one line per account with the username, UID, primary GID, home directory, and login shell. `/etc/group` only lists groups and their members."
      - q: "What does the read (`r`) permission allow on a **directory**?"
        options:
          - "Deleting the directory"
          - "Entering it with `cd`"
          - "Listing the files inside it"
          - "Running it as a program"
        answer: 3
        explanation: "Read on a directory means you can see the names inside (`ls`). Entering it with `cd` needs **execute** (`x`), and adding or removing files needs **write** (`w`)."
      - q: "Which command changes the permissions of a file?"
        options:
          - "chmod"
          - "chown"
          - "passwd"
          - "usermod"
        answer: 1
        explanation: "`chmod` (change mode) edits permissions. `chown` changes who owns the file, `passwd` sets a password, and `usermod` edits a user account."
      - q: "What does `sudo` do?"
        options:
          - "Switches your default shell"
          - "Deletes a user account"
          - "Shows who is logged in"
          - "Runs a single command with administrator privileges"
        answer: 4
        explanation: "`sudo` lets an allowed user run one command as root (or another user) without logging in as root. Your normal account stays normal the rest of the time."
      - q: "How many **primary** groups can a Linux user have?"
        options:
          - "None"
          - "Exactly one"
          - "As many as they want"
          - "Up to five"
        answer: 2
        explanation: "Every user has exactly one primary group (stored as a GID in `/etc/passwd`). Anything beyond that is a secondary group, and a user can have zero or more of those."
  posttest:
    title: "Post-test"
    questions:
      - q: "What does `chmod 750 report.sh` do?"
        options:
          - "Everyone can read and run it"
          - "Owner can read and write; group can read; others get nothing"
          - "Owner gets full access, group can read and run, others get nothing"
          - "Owner can read, group can write, others get full access"
        answer: 3
        explanation: "Split 750 into three digits: `7` = rwx (owner), `5` = r-x (group), `0` = --- (others). Remember 4 = read, 2 = write, 1 = execute, and add them up."
      - q: "You run `sudo usermod -G developers dave` and Dave suddenly loses his sudo rights. What happened?"
        options:
          - "`-G` replaced all of his secondary groups, so he was removed from `sudo`. `-aG` would have appended instead"
          - "The `developers` group blocks sudo by design"
          - "`usermod` always deletes the user's home directory"
          - "Dave needs to reboot for the group to apply"
        answer: 1
        explanation: "Without `-a`, the `-G` option **overwrites** the whole secondary group list. Always use `usermod -aG group user` when you only want to add a group."
      - q: "What does the setgid bit do on a shared directory?"
        options:
          - "Only the file owner can delete files inside"
          - "Anyone can run files inside as root"
          - "It hides the files from other users"
          - "New files inherit the directory's group instead of the creator's primary group"
        answer: 4
        explanation: "With setgid (`chmod g+s dir`), everything created inside gets the directory's group. That keeps shared folders consistent. The 'only the owner can delete' behavior belongs to the **sticky bit**."
      - q: "You have `r--` but no `x` on a directory. What happens when you run `cd` into it?"
        options:
          - "It works, read is enough"
          - "Permission denied, because `cd` needs execute"
          - "It asks for your sudo password"
          - "It works, but you can't change anything"
        answer: 2
        explanation: "On a directory, `x` is the permission that lets you pass through it. Without it you can't `cd` in or open anything inside, no matter what the `r` says."
      - q: "What is the safest way to give a coworker admin rights?"
        options:
          - "Share the root password with them"
          - "Let them log in with your account"
          - "Add their own account to the `sudo` (or `wheel`) group"
          - "Run `chmod 777 /etc`"
        answer: 3
        explanation: "Each person gets their own account, and admin rights come through `sudo`. That keeps the logs honest (you can see who ran what) and you can take the access back at any time."
---

{% include quiz.html name="pretest" %}

---

## **Who Is on This Machine?**

Linux was built as a multi-user system. Many people (and many services) can be logged in at once, so the OS needs a way to keep them out of each other's stuff. That's the whole job of **users, groups, and permissions**.

There are three kinds of accounts you'll run into:

| Type | UID (usually) | Notes |
|---|---|---|
| root | `0` | The superuser. Can do anything, no questions asked. |
| System users | `1` to `999` | Created for services like `www-data` or `postfix`. They normally can't log in (`/usr/sbin/nologin`). |
| Regular users | `1000` and up | Real people. These are the accounts you create. |

Account info lives in three plain text files:

| File | Contains |
|---|---|
| `/etc/passwd` | One line per user: name, UID, GID, home, shell |
| `/etc/shadow` | The password hashes (readable by root only) |
| `/etc/group` | One line per group and its members |

Here is one line from `/etc/passwd`, split by colons:

```
alice:x:1001:1001:Alice:/home/alice:/bin/bash
```

| Field | Value | Meaning |
|---|---|---|
| 1 | `alice` | Username |
| 2 | `x` | Password placeholder, the real hash is in `/etc/shadow` |
| 3 | `1001` | UID |
| 4 | `1001` | GID of the **primary group** |
| 5 | `Alice` | Comment / full name |
| 6 | `/home/alice` | Home directory |
| 7 | `/bin/bash` | Login shell |

```bash
# Who am I, and what groups do I have?
whoami
id

# Look up another user
id alice
getent passwd alice
```

---

## **Managing Users**

| Command | What it does |
|---|---|
| `sudo adduser alice` | Friendly wrapper on Debian/Ubuntu. Asks questions, creates the home directory and a group with the same name. |
| `sudo useradd -m -s /bin/bash alice` | The low-level, works-everywhere version. Forget `-m` and you may end up with no home directory. |
| `sudo passwd alice` | Set or change a password (run `passwd` alone to change your own). |
| `sudo usermod -s /bin/zsh alice` | Modify an existing account (shell, home, groups, and more). |
| `sudo passwd -l alice` | Lock an account without deleting it. |
| `sudo userdel alice` | Delete the account only. The home directory stays behind. |
| `sudo userdel -r alice` | Delete the account **and** its home directory. |

> **Good to know:** on Debian/Ubuntu use `adduser`, on RHEL/Rocky use `useradd`. Both end up in the same files, they just ask different amounts of questions.
{: .prompt-tip}

---

## **Groups**

A group is simply a named list of users. Instead of giving access to five people one by one, you give it to a group once and then move people in and out.

- **Primary group**: the one stored in `/etc/passwd`. Every user has exactly one, and every new file they create is tagged with it.
- **Secondary groups**: extras you add later. A user can have none, one, or many.

| Command | What it does |
|---|---|
| `groups alice` | Show all groups for a user |
| `id alice` | Same, but with the numeric IDs too |
| `sudo groupadd webteam` | Create a group |
| `sudo usermod -aG webteam alice` | **Add** alice to a secondary group |
| `sudo gpasswd -d alice webteam` | Remove alice from a group |
| `sudo groupdel webteam` | Delete a group |
| `newgrp webteam` | Start a shell that uses `webteam` as the active group |

> **Watch out:** `usermod -G` without `-a` **replaces** all secondary groups. It's an easy way to accidentally remove yourself from `sudo`. Always type `-aG`.
{: .prompt-warning}

> Group changes don't reach sessions that are already open. Log out and back in (or use `newgrp`) before testing.
{: .prompt-tip}

---

## **Sudo, Not Root**

Doing everything as a root all day is risky. One typo and there's no safety net. The usual approach is to work as a normal user and use admin power only when you need it, with `sudo`.

```bash
sudo apt update          # one command as root
sudo -l                  # list what you're allowed to run
```

Who gets to use sudo is decided by group membership:

| Distro family | Admin group |
|---|---|
| Debian / Ubuntu | `sudo` |
| RHEL / Rocky / Fedora | `wheel` |

```bash
sudo usermod -aG sudo alice      # Debian/Ubuntu
sudo usermod -aG wheel alice     # RHEL family
```

For finer rules, edit the sudoers file, **only** with `visudo`. It checks the syntax before saving, so a typo can't lock you out.

```bash
sudo visudo -f /etc/sudoers.d/alice
```

```
# alice may restart nginx and nothing else
alice ALL=(ALL) /usr/bin/systemctl restart nginx
```

> **Best practice:** give people the smallest set of commands they need (least privilege), and use full paths in sudoers (`which systemctl` tells you the path).
{: .prompt-tip}

---

## **Reading Permissions**

Run `ls -l` and look at the first column:

```
-rwxr-xr-- 1 alice webteam 220 Oct  4 10:00 deploy.sh
```

| Chars | `-` | `rwx` | `r-x` | `r--` |
|---|---|---|---|---|
| Meaning | Type (`-` file, `d` directory, `l` link) | **Owner** (alice) | **Group** (webteam) | **Others** (everyone else) |

Then comes the link count, owner, group, size, date, and name.

The same three letters mean different things on files and directories:

| | File | Directory |
|---|---|---|
| `r` read | View the contents | List the names inside |
| `w` write | Change the contents | Create, rename, or delete entries inside (needs `x` too) |
| `x` execute | Run it as a program | Enter it with `cd` and reach what's inside |

---

## **Changing Permissions**

### chmod

You can describe the change with letters or with numbers.

```bash
# Letters: who (u g o a) + operator (+ - =) + permission (r w x)
chmod u+x deploy.sh        # let the owner run it
chmod g+w,o-rwx notes.txt  # group can write, others get nothing
chmod -R g+rX project/     # recursive; capital X only adds x on directories

# Numbers (octal)
chmod 640 notes.txt
```

For the numeric style, each permission has a value, and you add them per column:

| Permission | Value |
|---|---|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |

So `rwx` = 4+2+1 = **7**, `r-x` = 4+1 = **5**, `rw-` = 4+2 = **6**, `r--` = **4**.

| Mode | Looks like | Typical use |
|---|---|---|
| `644` | `rw-r--r--` | Normal files |
| `755` | `rwxr-xr-x` | Scripts, public directories |
| `600` | `rw-------` | Private files like SSH keys |
| `640` | `rw-r-----` | Configs the group may read |
| `770` | `rwxrwx---` | Shared team folder |

> **Don't reach for `777`.** It means every user on the box can change the file. If something "needs 777 to work", the real fix is almost always the right owner or group.
{: .prompt-warning}

### chown and chgrp

```bash
sudo chown alice file.txt              # change owner
sudo chown alice:webteam file.txt      # owner and group together
sudo chown -R alice:webteam /srv/site  # whole tree
sudo chgrp webteam file.txt            # group only
```
---
 
## **umask: Default Permissions**
 
Brand new file is `644` and a new folder is `755`, without you ever running `chmod`? That's the **umask** at work.
 
Think of it as a list of things to **take away** from every new file or folder. Linux starts with the maximum, then the umask crosses permissions out.
 
| New item | Starts as |
|---|---|
| File | `666` (`rw-rw-rw-`), files never start with `x` |
| Directory | `777` (`rwxrwxrwx`) |
 
The most common umask is `022`. Read it as three digits, one each for owner, group, and others. A `2` means "remove write".
 
| | Owner | Group | Others |
|---|---|---|---|
| umask `022` | remove nothing | remove `w` | remove `w` |
| New file | `rw-` | `r--` | `r--` |
| New directory | `rwx` | `r-x` | `r-x` |
 
That gives you `644` for files and `755` for directories.
 
For the usual umask values, a quick shortcut works: subtract the umask from the starting mode. File: `666 - 022 = 644`. Directory: `777 - 022 = 755`.
 
| umask | Files | Directories | Use it when |
|---|---|---|---|
| `022` | `644` | `755` | Default. Others can look but not touch |
| `002` | `664` | `775` | Team folders where the group edits together |
| `077` | `600` | `700` | Private. Only the owner gets in |
 
```bash
umask          # show the current value
umask 002      # change it for this terminal session only
```
 
> The umask only affects files created **after** you set it. Existing files keep their permissions, and closing the terminal resets it. To keep it, add the `umask` line to `~/.bashrc`.
{: .prompt-tip}


---

## **Special Bits**

Three extra flags that show up in multi-user setups:

| Bit | Set with | On a file | On a directory |
|---|---|---|---|
| setuid | `chmod u+s` (4xxx) | Runs with the **owner's** rights (that's how `passwd` can edit `/etc/shadow`) | Ignored |
| setgid | `chmod g+s` (2xxx) | Runs with the group's rights | New files **inherit the directory's group** |
| sticky | `chmod +t` (1xxx) | Ignored | Only the file's owner (or root) can delete or rename it |

You've already seen the sticky bit: `ls -ld /tmp` shows `drwxrwxrwt`. Everybody can write there, but you can't delete someone else's files.

> setuid on your own scripts is a security risk. Leave it to system tools unless you really know why you need it.
{: .prompt-warning}

---

## **Good Habits**

| Do this | Because |
|---|---|
| One account per person | Files stay separated and logs show who did what |
| Never share passwords | Shared logins can't be audited or revoked cleanly |
| Use groups for teamwork | One change to the group, instead of editing every file |
| Use `sudo` instead of root | Fewer mistakes, better logging |
| Give the least access needed | A compromised account can only do so much |
| Review with `ls -l` and `id` | Check before you assume |

---

## **Key Commands Cheat Sheet**

| Command | What it does |
|---|---|
| `id`, `whoami`, `groups` | Who am I and what groups do I belong to |
| `adduser`, `useradd -m`, `userdel -r` | Create or remove a user |
| `passwd` | Set a password |
| `groupadd`, `usermod -aG`, `gpasswd -d` | Create a group, add or remove members |
| `sudo`, `visudo` | Run as admin, safely edit sudo rules |
| `ls -l`, `ls -ld` | Read permissions (use `-d` for a directory itself) |
| `chmod`, `chown`, `chgrp` | Change permissions, owner, group |
| `getent passwd`, `getent group` | Look up accounts and groups |

---

## **Hands-On Case**

**Scenario:** A small web team (Alice and Bob) needs one shared folder at `/srv/webteam`. They should both be able to edit each other's files. Carol works in another department and must stay out.

Try it yourself first, then open the solution to check.

**Goal:**
1. Create the group `webteam` and the users `alice`, `bob`, `carol`
2. Put only Alice and Bob in the group
3. Make `/srv/webteam` owned by `root:webteam`, with group-shared files and no access for others
4. Prove it works for Alice and Bob, and fails for Carol

<details markdown="1">
<summary>Show solution</summary>

```bash
# 1. Create the group and the users
sudo groupadd webteam
sudo useradd -m -s /bin/bash alice
sudo useradd -m -s /bin/bash bob
sudo useradd -m -s /bin/bash carol

# 2. Add Alice and Bob (Carol stays out)
sudo usermod -aG webteam alice
sudo usermod -aG webteam bob

# 3. Build the shared directory
sudo mkdir /srv/webteam
sudo chown root:webteam /srv/webteam
sudo chmod 2770 /srv/webteam

# 4. Check the setup
ls -ld /srv/webteam      # drwxrws---  (the "s" is the setgid bit)
id alice
id bob
```

```bash
# 5. Test it, one user at a time
 
# --- Alice creates a file ---
sudo su - alice                              # switch to Alice
whoami                                       # should print: alice
umask 002                                    # let the group edit new files
echo "<h1>Hello</h1>" > /srv/webteam/index.html
ls -l /srv/webteam                           # group is webteam, even though Alice's primary group is alice
exit                                         # go back to your own user
```
 
```bash
# --- Bob edits the same file ---
sudo su - bob                                # switch to Bob
whoami                                       # should print: bob
echo "<p>Bob was here</p>" >> /srv/webteam/index.html
cat /srv/webteam/index.html                  # you should see both lines
exit
```
 
```bash
# --- Carol tries to peek ---
sudo su - carol                              # switch to Carol
whoami                                       # should print: carol
ls /srv/webteam                              # Permission denied
exit
```

**Command Explanation**

| Part | What it does |
|---|---|
| `useradd -m -s /bin/bash` | Create the user with a home directory and a bash shell |
| `usermod -aG webteam` | Append the group, don't overwrite existing ones |
| `chown root:webteam` | Root owns the folder, the team group controls it |
| `chmod 2770` | `2` = setgid, then `rwx` owner, `rwx` group, `---` others |
| `sudo -u alice` | Run the command as Alice without needing her password |
| `umask 002` | Make new files group-writable (see the note below) |

> **Why the `umask 002`?** The default umask `022` creates files as `644`, so the group could read Alice's file but not edit it. Setting `002` gives new files `664`. In real setups you'd put this in the users' shell profile.
{: .prompt-tip}

```bash
# cleanup
sudo userdel -r alice
sudo userdel -r bob
sudo userdel -r carol
sudo groupdel webteam
sudo rm -rf /srv/webteam
```

> **Going further:** add the sticky bit (`chmod 3770 /srv/webteam`) so Bob can edit Alice's files but can't delete them.
{: .prompt-tip}

</details>

---

## **Glossary**

| Term | Detail |
|---|---|
| UID | Numeric ID of a user. root is always `0` |
| GID | Numeric ID of a group |
| Primary group | The one group stored in `/etc/passwd`; new files get tagged with it |
| Secondary group | Any extra group a user is added to |
| sudo | Tool that runs one command with elevated privileges |
| Octal mode | Permissions written as numbers (`4`=r, `2`=w, `1`=x), like `750` |
| setgid | Bit that makes new files in a directory inherit its group |
| Sticky bit | Bit that stops users from deleting each other's files in a shared directory |
| umask | The default "subtract these permissions" mask applied to new files |

---

## **Finish**

Thanks for following along. Before moving on, run through the post-test below and see how far you've come.

---

{% include quiz.html name="posttest" compare="pretest" %}