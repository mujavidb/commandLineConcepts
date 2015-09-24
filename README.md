# Things I didn't understand when starting off backend dev

When I first began using the command line for web developement there were a number of things I didn't understand. For a time, I avoided a lot of these as I didn't understand them and hadn't come across any guides explaining any of them in a holistic sense.

This guide is not a tutorial. It is here to explain and provide background information on common web dev tasks done on the command line.

A lot of the things discussed here will relate to both Mac OS and Linux, however, I will be focussing solely on Linux.

This guide assumes familiarity with the following commands and operations:
* `ls`
* `cd`
* `mkdir`, `cp`, `mv` and `rm`
* `man`
* `>`, `>>`, `<` and `|`

This guide isn't made to be consumed at once, but it's here to expand on any of the following if you need to:
* [The Linux Filesystem](#linuxFiles)
* [File access and ownership](#fileAccess)
* [`.bashrc` and environment variables (PATH/HOME)](#bashrc)
    - [Environmental variables](#envVars)
    - [Aliases](#aliases)
    - [Dotfiles](#dotfiles)
* [SSH Keys](#ssh)
* [Symbolic links](#symbolicLinks)

## [linuxFiles]The Linux Filesystem

All Linux OS distributions are based on the UNIX file system.

The highest directory is called root and it's path is `/`. Inside the root are several subdirectories each with their own uses.

```
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
When a user logs in to the server their login location will be in a directory inside `/home`, commonly `/home/USERNAME`. This is where a user will usually store files belonging to them such as colorschemes, ssh keys and config files. This will also be where the path `~/` leads to for the current logged user.

#### `/root`
Unlike other users the root user has their own home directory located in the root folder. These two are not to be confused; the root directory `/` and the root user directory `/root` are different locations.

#### `/srv`
This is where your website will stay. You can choose to place your site directly inside it or in a subdirectory, it's up to you.

#### `/var`
This directory is for variable data files, this includes files like logs, backups and temps.

#### `/usr`
This folder should be used for shareable, read-only data that supports user applications. This includes program manuals, documentation files and dictionaries for spelling checkers. You won't be using this directory much aside from the odd tutorial.

#### `/etc`
This directory contains the most important system configuration files. These files include things like your `php.ini`, `nginx.conf` and `apache2.conf` files. They are very important and you will be fiddling with them a lot to get things working right.

#### `/bin`, `/sbin`, `/usr/bin`, `/usr/local/bin` and `/usr/sbin`
These four directories contain programs used by the system and its users. When you download programs to your server they will be saved in one of these directories.
`/bin` contains essential programs that the system requires to operate such as `ls` and `ln`. 
`/usr/bin` contains programs for the systems normal users such as `git` and `wget`.
`/usr/local/bin` contains programs that do not come as part of the official distribution such as `pip` and `bundle`.
`/sbin` and `/usr/sbin` both contain programs used by system administrators such as `reboot` and `adduser`.
It's important to note that the programs at these locations can all be run with their direct path e.g. `/bin/ls -al` instead of `ls -al`.

## [fileAccess]File access and ownership

All users belong to a user group. A user can belong to multiple groups, but they will always have one group as their __primary group__. The primary group is the group applied to you when you log in to the shell. Each group can have multiple users.

All files and directories can have three actions carried out on them, these are: __read__, __write__ and __execute__. These can be written as `rwx` and are represented in binary:
* `--x` as 001 which in binary is 1
* `-w-` as 010 which in binary is 2
* `-wx` as 011 which in binary is 3
* `r--` as 100 which in binary is 4
* `r-x` as 101 which in binary is 5
* `rw-` as 110 which in binary is 6
* `rwx` as 111 which in binary is 7

These actions can be limited to specific users and user groups, for security reasons. There are three levels on which the above access permissions can be set: __the file owner__, __the file owner's group__ and __all other users__.

Here are a couple of examples:
* `755` as `rwxr-xr-x`, this would give the file owner `rwx` permission on the file, however, the file owner's group and all other users would only have `r-x` permission.
* `644` as `rw-r--r--`, this would give the owner `rw-` and `r--` for all other users.

If I run the command `ls -Al` (list with hidden files and no `.` directories) in my `.ssh` directory I get the following:
```
total 80
-rw------- 1 webmaster webmaster 801 Sep 18 11:41 authorized_keys
```

The `d` or `-` at the start of each line shows that it is either a directory or a file. The next nine characters tell us the file permissions `rw-------` .The first `www-data` is the user to which the file belongs. The second `www-data` is the file owner's __primary group__ and thus only that group will get the file or directory's group permissions.

We can modify permissions with `chmod`, ownership with `chown` and group with `chgrp`.
```bash
$ ls -Al
drwxr-xr-x 1 root root 801 Sep 18 11:31 directory1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber2
$ chmod 644 filenumber1
$ ls -Al
drwxr-xr-x 1 root root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 root root 801 Sep 18 11:31 filenumber2
$ chown webmaster filenumber2
$ ls -Al
drwxr-xr-x 1 root      root 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root      root 801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 webmaster root 801 Sep 18 11:31 filenumber2
$ chgrp www-data directory1
$ ls -Al
drwxr-xr-x 1 root      www-data 801 Sep 18 11:31 directory1
-rw-r--r-- 1 root      root     801 Sep 18 11:31 filenumber1
-rwxr-xr-x 1 webmaster root     801 Sep 18 11:31 filenumber2
```

It is important to note that the root user has `rwx` permissions for all files. To carry out a command specifically as the root user we would prepend a command with the `sudo` command.

## [bashrc].bashrc and environment variables

`.bashrc` is a configuration file that is executed every time you open/log into a terminal. `.bashrc` contains settings for useful things such as environment variables, aliases and scripts. If you have made changes to your `.bashrc` you can have them applied in your current shell instance with the commans `source ~/.bashrc`, depending on where your `.bashrc` is located.

### [envVars]Environment variables
$PATH is an environment variable that specifies directories where executable programs like `ls` and `cd` are located. It is because of the PATH variable that the computer knows where to find programs when you execute them. So instead of typing `/bin/ls` you can type `ls`. 

You will often see something like this in your `~/.bashrc`, with each path seperated by a colon `:`
```bash
export PATH="/usr/local/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
```

The `export` command sets environment variables. To add a new path to the `PATH` variable we would add `export PATH=/home/user/.rbenv/bin` to the end of our `.bashrc`. 

This however will completely overrite the PATH variable, thus, to retain any previous values in PATH we would add `$PATH` to the end, like so `export PATH=/home/user/.rbenv/bin:$PATH`.

For non-root users, `.` is usually included in `PATH` so that you can run programs from the current directory. This is discouraged for the root user incase a malicious program/tarbomb is accidentally run. The path for the root user will also contain `/sbin` and `/usr/sbin` for the use of adminstrative programs.

You can quickly view all environment variables with the command `env`.

### [aliases]Aliases
Aliases allow a user to set custom commands for programs. Here are some examples of aliases that have been added to a `.bashrc`.
```bash
alias ll="ls -l"
alias la="ls -alh"
alias ..="cd .."
alias ...="cd ../.."
```

Like `export`, `alias` can also be run on the command line, however, its changes will only last for the current session.

### [dotfiles]Dotfiles
Dotfiles are files that begin with a dot and are hidden by default. Most commonly, they are files that contain settings for your bash environment including custom functions, aliases and other settings that come in handy when working on the command line.

Saving your aliases and other custom functions outside of `.bashrc` will decouple the code and is considered good practice. 

Below is a basic dotfile I've created.
__`.my_dot_file`__:
```bash
#!/bin/bash
# My first dotfile

alias mycloud="ssh webmaster@mywebsite.com"
```

If I want to execute this file when a bash instance begins (so that I have access to the alias) I can source it in my `.bashrc` like so: `source ~/.my_dot_file`

Many developers [share their dotfiles online](https://dotfiles.github.io/).It's great to see the cool customizations the community comes up with and incorporate it into your own workflow. Bear in mind, it's important to understand what the code in your dotfiles does and to tailor it to your needs.

## [ssh]SSH Keys

When you access your server from your local machine via the command line you will often use the `ssh` command, e.g. `ssh user@mysite.com` or `ssh user@47.102.0.218`. You will then be prompted for a password and on correct entry will be logged in to your remote machine via the command line.

Using a password by itself for authentication is not secure and should be avoided. This is where SSH keys come in.

After generating an SSH key on your local machine, you can place its public version in your remote machine's `~/.ssh/authorized_keys` file. When you next `ssh` into your remote machine you will no longer be asked for a password as authentication will take place via your ssh keys.

The basic commands for this will be:
```bash
$ ssh-keygen 
$ # this will generate the key in your ~/.ssh directory

$ cat ~/.ssh/id_rsa.pub | ssh user@remotemachine "mkdir ~/.ssh; cat >> ~/.ssh/authorized_keys"
$ # replace id_rsa with your file name
$ # this will append your ssh-key to the end of your authorized_keys file
```

## [symbolicLinks]Symbolic links
We can use the `ln` command to create links between files and directories. There are two types of links, hard links or soft links. We will focus entirely on softlinks also known as symbolic links or symlinks.

Symbolic links act as shortcuts to other files. We use the command `ln -s` to create a link in one location to a file or directory in another location. The syntax works like this: `ln -s target_path link_path`

Let's look at the example below.
```bash
$ pwd
/home/mujavidb/newdir
$ ls
$ ln -s ~/.vimrc mylink
$ ls -Al
lrwxr-xr-x  1 webmaster  staff  18 24 Sep 15:10 mylink -> /home/mujavidb/.vimrc
```

After running `ln -s` I created a new file called `mylink`. You can tell this is a link because the line begins with an `l` instead of a `d` or `-`. You can also see the original file which it links back to. If I chose to open the file `mylink` the file `.vimrc` would open.
