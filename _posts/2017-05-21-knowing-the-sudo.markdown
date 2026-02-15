---
title: "Knowing the Sudo"
layout: post
date: 2017-05-21 18:48
tag:
- linux
- sudo
description: The obscure facts about sudo
---

Among the most frequently used commands in Linux is `sudo`.

Any ***permission denied*** message on the CLI instinctively tickles us to prepend sudo and feel powerful. But there's more to it than just elevated permissions. Here are some concepts one should know for efficient use and to play safe with it.

Let's look at a simple example involving environment variables.

```bash
[vagrant@localhost ~]# export DUMMY=dummy
[vagrant@localhost ~]# env | grep DUMMY
DUMMY=dummy
[vagrant@localhost ~]# sudo env | grep DUMMY

[vagrant@localhost ~]#
```
Different settings and context switches that take place in the background when invoking sudo can easily be overlooked, leading you astray while troubleshooting.

As you can see below, running node as sudo invokes a different environment entirely. The user would expect the `SERVICE_ENV` variable to be `qa`, but it's undefined.

```bash
[vagrant@localhost ~]# node
> process.env.SERVICE_ENV
'qa'
[vagrant@localhost ~]# sudo node
> process.env.SERVICE_ENV
undefined
```

## What is *sudo*?

>***sudo*** - "substitute user do" or "super user do" is used to execute commands with elevated privileges (usually root)

#### Ways of becoming root

There are multiple ways to become root on a Unix-based system.
1. sudo -s
2. su
3. su -
4. sudo -i
5. sudo su -
6. sudo su

Ok! Now, what is `su`?

>***su***, or "switch user", lets you run a command as another user (usually root)

## Differences between *sudo* and *su*
* `su` requires the password of the *target* user, whereas `sudo` requires the password of the user who invoked it. This matters on shared servers, where sharing the root password is dangerous. `sudo` lets users elevate using their own password.

* `su` invokes a shell, allowing multiple commands, whereas `sudo` executes a single command with elevated privileges.

* The user who invoked `sudo` can be tracked, whereas once someone becomes root via `su`, it's difficult to trace who did what.

```bash
[vagrant@localhost ~]$ su
Password:
[root@localhost vagrant]# pwd
/home/vagrant
```

The `-` (hyphen) argument passed to `su` creates a fresh login environment (as defined by root's `.bashrc`), analogous to logging in directly as root.

```bash
[vagrant@localhost ~]$ su -
Password:
Last login: Sat May 20 08:43:26 UTC 2017 on pts/0
[root@localhost ~]# pwd
/root
```

The different methods of becoming root primarily differ in the `$HOME` directory and the environment they inherit.

A user's environment is typically defined in their `.bashrc`.

| Command | $HOME | Env |
| :------------- | :------------- |:------------- |
| sudo -i | root | root |
| sudo -s | USER | USER |
| sudo /bin/bash | USER | USER |
| sudo su | root | USER |
| sudo su - | root | root |

## Controlling sudo privileges to a user

A user's sudo privileges are controlled by `/etc/sudoers`.

Editing this file directly with vi or any editor is dangerous. A broken sudoers file could mean a broken system (fixable only by booting into rescue mode).

> **visudo** is a utility to edit the sudoers file safely. It validates syntax before saving and discards changes if errors are found.

> Another option is to add a file with custom privileges for a particular user in the `/etc/sudoers.d/` directory.

This is especially convenient when using configuration management tools like Chef, since adding a file is simpler than editing `/etc/sudoers` directly.

The basic syntax in the file would be:

```
USER PLACES=(AS_USER) [NOPASSWD:] COMMAND
```

which translates to: "USER" can execute "COMMAND" in "PLACES" as "AS_USER" without a password.
Each of these fields can be an alias referring to a group of entities, like `User_Alias`, `Cmnd_Alias`, etc.

Example:
```
User_Alias USERS = tom, dick, harry
```

```
root ALL=(ALL)  ALL
```
This says: the user root can execute ALL commands as ALL users from ALL places.

### Some useful flags in sudoers

#### Ever missed the asterisk while entering sudo password?

Seeing asterisks while typing your sudo password helps you know how many characters you've entered, which is useful for catching mistakes. Enable it with the **pwfeedback** flag.

#### Ever noticed that sudo doesn't ask for your password again for a while?

Once authenticated, sudo remembers your credentials for 15 minutes by default. This can be changed with **timestamp_timeout=[value]**, where value is in minutes.

```
[root@layer201 vagrant]# grep 'pwfeedback' /etc/sudoers
Defaults    env_reset,pwfeedback,timestamp_timeout=10

[vagrant@layer201 ~]$ sudo -s
Password: *******
```

That's it! Thanks for reading.
