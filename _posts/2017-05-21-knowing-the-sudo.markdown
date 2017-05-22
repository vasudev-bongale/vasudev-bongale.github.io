---
title: "Knowing sudo"
layout: post
date: 2017-05-21 18:48
headerImage: false
tag:
- linux
- sudo
category: blog
author: vasudev
description: The obscure facts about sudo
---

Among the most frequent commands that a Linux user uses is the "sudo" command.

Any ***permission denied*** message on the CLI effortlessly tickles us to append sudo and feel powerful. Here are some of the concepts around the 'SUDO' illustrating beyond its ability to provide elevated permissions, that one should know for efficient use and to play safe with it.

Lets look at a simplest example of dealing with a variable.

```bash
[vagrant@localhost ~]# export DUMMY=dummy
[vagrant@localhost ~]# env | grep DUMMY
DUMMY=dummy
[vagrant@localhost ~]# sudo env | grep DUMMY

[vagrant@localhost ~]#
```
Different settings or contexts switches that takes place in the background along with invocation of sudo could potential be overlooked, thus deviating from troubleshooting any issue with your expected workflow.

As you can see below, running node as sudo might invoke a totally different environment, whereas the user would expect 'qa' value to the variable.   

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

There are multiple ways one can become a root user in Unix based system.
1. sudo -s
2. su
3. su -
4. sudo -i
5. sudo su -
6. sudo su

Ok! Now, what is 'su' !?

>***su*** can be equivalently read as 'switch user', helps you to run a command as another user (usually root)

## Differences between *sudo* and *su*
* 'su' requires you to enter the password of the target user, whereas sudo requires the password of user who invoked 'sudo'
As is it clear, if su is used to switch to root, you will need the root password. This is not feasible in a shared server, wherein sharing the root password to users could be dangerous. Instead 'sudo' makes it easier for users to switch to root using there own password.

* 'su' invokes a shell, allowing the user to run multiple commands, whereas sudo can be used to execute a single command with elevated privileges.

* The user who invoked 'sudo' can be tracked, whereas once the user becomes root using 'su', its difficult to identify the user for tracing.

```bash
[vagrant@localhost ~]$ su
Password:
[root@localhost vagrant]# pwd
/home/vagrant
```

'-', the hyphen argument passed to 'su' will allow the user to create a new environment (usually defined by root's .bashrc), analogous to loggin in directly as root.

```bash
[vagrant@localhost ~]$ su -
Password:
Last login: Sat May 20 08:43:26 UTC 2017 on pts/0
[root@localhost ~]# pwd
/root
```

The different methods of becoming root have their primary differences in the $HOME(login directory) and the Environment after the login.

The environment of a user is assumed to be defined in its .bashrc.

| Command | $HOME | Env |
| :------------- | :------------- |:------------- |
| sudo -i | root | root |
| sudo -s | USER | USER |
| sudo /bin/bash | USER | USER |
| sudo su | root | USER |
| sudo su - | root | root |

## Controlling sudo privileges to a user

Any user's sudo privileges are controlled by file /etc/sudoers.

Editing this file directly using vi or any editor is dangerous, a broken sudoers file could mean a broken system (which can be fixed by booting the server in rescue mode)

> visudo - a utility to edit sudoers file safely with benefit of discarding any changes with incorrect syntax. It also takes a backup of the current file before allowing the user to edit.

> Another easy option to have modular control is to add a file with custom privileges for a particular user in /etc/sudoers.d/ directory.

This is an easy option if a configuration management tools such as chef is used to control the access. Adding a file would be more easy than editing the /etc/sudoers file.

The basic syntax in the file would be:

```
USER PLACES=(AS_USER) [NOPASSWD:] COMMAND
```

which translates to "USER" can execute "COMMAND" in "PLACES" as "AS_USER" without password.
Each of these fields can be an aliases to set/group of entities, like User_Alias, Cmnd_Alias etc.

Example:
```
User_Alias USERS = tom, dick, harry
```

```
root ALL=(ALL)  ALL
```
saying, The user root can execute ALL commands as ALL users from ALL places.

### Some useful flags in sudoers

#### Ever missed the asterisk while entering sudo password?

The asterisk characters while entering sudo password might be helpful when you entered the password wrong and you need to clear the buffer to re-enter. It can be easily enabled by adding a default flag - **pwfeedback**

#### Ever noticed that sudo does not ask password once entered for sometime?

The default session duration for which the passwords are bypassed once successfully authenticated is 15 minutes. This can be overwritten by a flag - **timestamp_timeout=[value]**, where value is in minutes.

```
[root@layer201 vagrant]# grep 'pwfeedback' /etc/sudoers
Defaults    env_reset,pwfeedback,timestamp_timeout=10

[vagrant@layer201 ~]$ sudo -s
Password: *******
```

Thats it! Thanking for reading!
Comment down what you think.
