# New concepts to learn when using linux on command line

When I first began using the command line for web developement there were a number of things I didn't understand. For a time, I avoided learning about these as I didn't understand them and hadn't come across any guides explaining any of them in a holistic sense.

This guide is not a tutorial. It is here to explain and provide background information on common web developement tasks done on the command line.

A lot of the things discussed here will relate to both Mac OS and Linux, however, I will be focussing solely on Linux in a server environment.

This guide assumes familiarity with the following commands and operations:
* `ls`
* `cd`
* `mkdir`, `cp`, `mv` and `rm`
* `man`
* `>`, `>>`, `<` and `|`

This guide isn't made to be consumed at once, but it's here to expand on any of the following:
* [The Linux Filesystem](#linuxFiles)
* [File access and ownership](#fileAccess)
* [.bashrc and environment variables (PATH/HOME)](#bashrc)
    - [Environmental variables](#envVars)
    - [Aliases](#aliases)
    - [Dotfiles](#dotfiles)
* [SSH Keys](#ssh)
* [Symbolic links](#symbolicLinks)

## <a name="linuxFiles"></a>The Linux Filesystem

All Linux OS distributions are based on the UNIX file system.

In a UNIX file system files can be grouped together to form directories, these are also known as folders. The highest directory is called root and it's path is `/`. Inside the root are several subdirectories each with their own uses.

```
$ ls
bin/   dev/  home/    lib/         misc/  opt/     root/  tmp/  var/
boot/  etc/  initrd/  lost+found/  mnt/   proc/    sbin/  usr/  srv/
```

The most commonly used directories among these are:

* `/home`
* `/root`
* `/var`
* `/etc`
* `/usr`
* `/srv`
* `/bin`, `/sbin`, `/usr/bin`, `/usr/local/bin` and `/usr/sbin`

#### `/home`
When a user logs in to the server their login location will usually be in a directory inside `/home`, commonly `/home/USERNAME`. This is where a user will usually store files belonging to them such as colorschemes, ssh keys and config files. This will also be where the path `~/` leads to for the current user.

#### `/root`
Unlike other users the root user has their own home directory located in the root folder. These two are not to be confused; the root directory `/` and the root user directory `/root` are different locations.

#### `/srv`
This is where your website will stay. You can choose to place your site directly inside it or in a subdirectory, it's up to you.

#### `/var`
This directory is for variable data files, this includes files like logs, backups and temp files.

#### `/usr`
This folder should be used for shareable, read-only data that supports user applications. This includes program manuals, documentation files and dictionaries for spelling checkers. You won't be using this directory much aside from the odd tutorial.

#### `/etc`
This directory contains the most important system configuration files. These files include things like your `php.ini`, `nginx.conf` and `apache2.conf` files. They are very important and you will be fiddling with them a lot to get things working right.

#### <a name="programDirs"></a>`/bin`, `/sbin`, `/usr/bin`, `/usr/local/bin` and `/usr/sbin`
These four directories contain programs used by the system and its users. When you download programs to your server they will be saved in one of these directories.

For a quick breakdown of what each folder contains:
* `/bin` contains essential programs that the system requires to operate such as `ls` and `ln`. 
* `/usr/bin` contains programs for the system's normal users such as `git` and `wget`.
* `/usr/local/bin` contains programs that do not come as part of the official distribution such as `pip` and `bundle`.
* `/sbin` and `/usr/sbin` both contain programs used by system administrators such as `reboot` and `adduser`.

It's important to remember that the programs at these locations can all be run with their direct paths e.g. `ls` can also be written as `/bin/ls`, as the `ls` program exists int the `/bin` directory.

## <a name="fileAccess"></a>File access and ownership

All users belong to at least one user group. A user can belong to multiple groups, but they will always have one group as their __primary group__. The primary group is the group applied to you when you log in to the shell.

The level of access a user can have for individual files and directories is outlined in that file or directory's permissions. 

All files and directories can have three actions carried out on them, these are: __read__, __write__ and __execute__. These actions can also be written as `rwx`. If a file only has read access this can be written as `r--` access.Likewise, files can have many different combinations of access. These can be represented in binary:
* `--x` as 001 which in binary is 1
* `-w-` as 010 which in binary is 2
* `-wx` as 011 which in binary is 3
* `r--` as 100 which in binary is 4
* `r-x` as 101 which in binary is 5
* `rw-` as 110 which in binary is 6
* `rwx` as 111 which in binary is 7

There are three levels on which the above access permissions can be set: __the file owner__, __the file owner's group__ and __all other users__.

Here are a couple of examples:
* `755` as `rwxr-xr-x`: this would give the file owner `rwx` permission on the file, however, the file owner's group and all other users would only have `r-x` permissions.
* `644` as `rw-r--r--`: this would give the owner `rw-` and `r--` for all other users.

If I run the command `ls -Al` (list with hidden files and no `.` directories) in my `.ssh` directory I get the following:
```
total 80
-rw------- 1 webmaster admins 801 Sep 18 11:41 authorized_keys
```

The `d` or `-` at the start of each line shows that it is either a directory or a file. The next nine characters tell us the file permissions `rw-------` . `webmaster` is the user to which the file belongs. `admins` is the file owner's __primary group__ and thus only that group will get the file or directory's group permissions.

We can modify permissions with `chmod`, ownership with `chown` and group with `chgrp`.
There are three commands that are used to modify permissions and ownerships:
* `chmod` changes permissions for the file
* `chown` changes the owner of the file
* `chgrp` changed the group of the file

```
$ ls -Al
drwxr-xr-x 1 root root 801 Sep 18 11:31 directory1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber2
$ chmod 644 filenumber1
$ ls -Al
drwxr-xr-x 1 root root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber2
```

You will see now that for `filenumber1` the permissions have changed from `rwxr-xr-x` to `rw-r--r--`. The level of access for all users has become more constricted as now the user only has `rw-` permission and all other users `r--` permission.

```
$ ls -Al
drwxr-xr-x 1 root root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber2
$ chown webmaster filenumber2
$ ls -Al
drwxr-xr-x 1 root      root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root      root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 webmaster root 801 Sep 18 11:31 filenumber2
```

After running the `chown` command the owner of the file `filenumber2` is now `webmaster`. Since `filenumber2` has `rwxr-xr-x` permissions, this now means the user `webmaster` now has `rwx` access to it.

```
$ ls -Al
drwxr-xr-x 1 root      root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root      root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 webmaster root 801 Sep 18 11:31 filenumber2
$ chgrp admins directory1
$ ls -Al
drwxr-xr-x 1 root      admins  801 Sep 18 11:31 directory1
-rw-r--r-- 1 root      root    801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 webmaster root    801 Sep 18 11:31 filenumber2
```

After running `chgrp`, the group of `directory1` has changed from `root` to `admins`. Since `directory1` has `rwxr-xr-x` permissions, this means that all users in the group `admins` that have it as their primary group now have `r-x` permissions to the directory.

It is important to note that the root user has `rwx` permissions for all files. For this reason it is recommend not to use the root user account as it is very easy to accidentally changes that are dangerous to your server.

For tasks that require root priveledges, it is preferred to only give root priveledges to specific commands. To carry out a command specifically as the root user we would prepend a command with `sudo`. The command `sudo mkdir new_stuff` will create the directory `new_stuff` and it's owner and group will both be set to `root`.

LinuxCommand has a guide that [talks a bit more about permissions](http://linuxcommand.org/lc3_lts0090.php).

## <a name="bashrc"></a>.bashrc and environment variables

`.bashrc` is a configuration file that is executed every time you open/log into a terminal. `.bashrc` contains settings for useful things such as environment variables, aliases and scripts. 

If you have made changes to your `.bashrc` you can have them applied in your current shell instance with the command `source ~/.bashrc`, depending on where your `.bashrc` is located.

### <a name="envVars"></a>Environment variables
$PATH is an environment variable that specifies directories where executable programs like `ls` and `cd` are located. It is because of the PATH variable that the machine knows where to find programs when you execute them. So instead of typing `/bin/ls` you can type `ls`. 

To view all directories listed in your path you can use the following command:
```
$ env | grep PATH
PATH="/usr/local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
```

The `PATH` variable shown above contains the paths to all directories that contain programs [mentioned above](#programDirs).

You will often see something like this in your `~/.bashrc`, with each path seperated by a colon `:`
```
...
export PATH="/usr/local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
```

The `export` command sets environment variables. To add a new path to the `PATH` variable we would add `export PATH=/home/user/.rbenv/bin` to the end of our `.bashrc`. 

This however will completely overrite the PATH variable, thus, to retain any previous values in PATH we would add `$PATH` to the end, like so `export PATH=/home/user/.rbenv/bin:$PATH`.

After that running the following command shows how the `PATH` variable has changed.

```
$ env | grep PATH
PATH=/home/user/.rbenv/bin:/usr/local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

For non-root users, `.` is usually included in `PATH` so that you can run programs from the current directory. This is discouraged for the root user incase a malicious program/tarbomb is accidentally run. 

The path for the root user will also contain `/sbin` and `/usr/sbin` for the use of adminstrative programs.

You can quickly view all environment variables with the command `env`.

### <a name="aliases"></a>Aliases
Aliases allow a user to set custom commands for programs. Here are some examples of aliases that have been added to a `.bashrc`.

```
alias ll="ls -l"
alias la="ls -alh"
alias ..="cd .."
alias ...="cd ../.."
```

Like `export`, `alias` can also be run on the command line, however, its changes will only last for the current session, if they are not inside `.bashrc`.

### <a name="dotfiles"></a>Dotfiles
Dotfiles are files that begin with a dot and are hidden by default. Most commonly, they are files that contain settings for your bash environment including custom functions, aliases and other settings that come in handy when working on the command line.

Saving your aliases and other custom functions outside of `.bashrc` will decouple the code and is considered good practice. 

Below is a basic dotfile I've created.
__`~/.my_dot_file`__:
```
#!/bin/bash
# My first dotfile

alias mycloud="ssh webmaster@mywebsite.com"
```

If I want to execute this file when a bash instance begins (so that I have access to the alias) I can source it in my `.bashrc` like so: `source ~/.my_dot_file`

Many developers [share their dotfiles online](https://dotfiles.github.io/).It's great to see the cool customizations the community comes up with and incorporate it into your own workflow. Bear in mind, it's important to understand what the code in your dotfiles does and to tailor it to your needs and preferences.

## <a name="ssh"></a>SSH Keys

When you access your server from your local machine via the command line you will often use the `ssh` command, e.g. `ssh user@mysite.com` or `ssh user@47.102.0.218`. You will then be prompted for a password and on correct entry will be logged in to your remote machine via the command line.

Using a password by itself for authentication is not secure and should be avoided. This is where SSH keys come in.

After generating an SSH key on your local machine, you can place its public version in your remote machine's `~/.ssh/authorized_keys` file. When you next `ssh` into your remote machine you will no longer be asked for a password as authentication will take place via your ssh keys.

To create an SSH key the basic command will be:
```
$ ssh-keygen 
```
You can follow the prompt to add a password to your key, but this isn't necessary. The command will generate two files in your `~/.ssh` directory. The first file will be a private key and the second a public key. By default these two files will be named `id_rsa` (private) and `id_rsa.pub` (public). Unless you know what you are doing, you should not be sending the private key to others as this is akin to giving your personal password away.

To access your remote machine via SSH, the public key must be copied and added to the file `~/.ssh/authorized_keys` on your remote machine. 

In the command below, replace `user` and `remotemachine` with the correct names. If you called your ssh key something different, replace `id_rsa` with the name you chose. After entering this command, you will be prompted for the password for your account on the remote machine. On entry, the public key will be copied over.

```
$ cat ~/.ssh/id_rsa.pub | ssh user@remotemachine "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

For more information on SSH setup, [check out this Digital Ocean article](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys).
To really understand the SSH Encryption and Connection process, [see this article](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process).

## <a name="symbolicLinks"></a>Symbolic links
We can use the `ln` command to create links between files and directories. There are two types of links, hard links and soft links. We will focus entirely on softlinks which are also known as symbolic links or symlinks.

Symbolic links act as shortcuts to other files. We use the command `ln -s` to create a link in one location to a file or directory in another location. The syntax works like this: `ln -s target_path link_path`

Let's look at the example below.
```
$ pwd
/home/mujavidb/newdir
$ ls
$
```

My working directory is currently empty. I will now create a link to my `~/.bashrc` in the current directory and call it `mylink`.

```
$ ln -s ~/.bashrc mylink
$ ls -Al
lrwxr-xr-x  1 webmaster  staff  18 24 Sep 15:10 mylink -> /home/mujavidb/.bashrc
```

After running `ln -s` I created a new file called `mylink`. After running the `ls` command, you can tell this is a link because the line begins with an `l` instead of a `d` or `-`. You can also see the original file which it links back to. 

If I chose to read, write or execute the file `mylink` the action would instead occur on `~/.bashrc`. However, if I change the location or delete `mylink` the change would happen to the symbolic link file and not on `~/.bashrc`.
