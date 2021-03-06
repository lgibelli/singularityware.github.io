---
title: Image Environment
sidebar: admin_docs
permalink: image-environment
folder: docs
---

## Directory access
By default Singularity tries to create a seamless user experience between the host and the container. To do this, Singularity makes various locations accessible within the container automatically. For example, the user's home directory is always bound into the container as is /tmp and /var/tmp. Additionally your current working directory (cwd/pwd) is also bound into the container iff it is not an operating system directory or already accessible via another mount. For almost all cases, this will work flawlessly as follows:

```bash
$ pwd
/home/gmk/demo
$ singularity shell container.img 
Singularity/container.img> pwd
/home/gmk/demo
Singularity/container.img> ls -l debian.def 
-rw-rw-r--. 1 gmk gmk 125 May 28 10:35 debian.def
Singularity/container.img> exit
$ 
```

For directory binds to function properly, there must be an existing target endpoint within the container (just like a mount point). This means that if your home directory exists in a non-standard base directory like "/foobar/username" then the base directory "/foobar" must already exist within the container.

Singularity will not create these base directories! You must enter the container with the `--writable` option being set, and create the directory manually.

### Current Working Directory
Singularity will try to replicate your current working directory within the container. Sometimes this is straight forward and possible, other times it is not (e.g. if the base dir of your current working directory does not exist). In that case, Singularity will retain the file descriptor to your current directory and change you back to it. If you do a 'pwd' within the container, you may see some weird things. For example:

```bash
$ pwd
/foobar
$ ls -l
total 0
-rw-r--r--. 1 root root 0 Jun  1 11:32 mooooo
$ singularity shell ~/demo/container.img 
WARNING: CWD bind directory not present: /foobar
Singularity/container.img> pwd
(unreachable)/foobar
Singularity/container.img> ls -l
total 0
-rw-r--r--. 1 root root 0 Jun  1 18:32 mooooo
Singularity/container.img> exit
$ 
```

But notice how even though the directory location is not resolvable, the directory contents are available.


## Standard IO and pipes

Singularity automatically sends and receives all standard IO from the host to the applications within the container to facilitate expected behavior from the interaction between the host and the container. For example:

```bash
$ cat debian.def | singularity exec container.img grep 'MirrorURL'
MirrorURL "http://ftp.us.debian.org/debian/"
$ 
Making changes to the container (writable)
By default, containers are accessed as read only. This is both to enable parallel container execution (e.g. MPI). To enter a container using exec, run, or shell you must pass the --writable flag in order to open the image as read/writable.
```

## Containing the container
By providing the argument `--contain` to `exec`, `run` or `shell` you will find that shared directories are no longer shared. For example, the user's home directory is writable, but it is non-persistent between non-overlapping runs.
