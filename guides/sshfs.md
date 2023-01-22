With the new computing system on SDF and AFS being depreciated, if you would like to mount your $HOME remotely (i.e. access it locally from your computer, for example to edit code in your favorite code editor), follow these instructions.

# 0. Install Meson and Ninja
[Meson](https://mesonbuild.com/) and [Ninja](https://ninja-build.org/) are new open-source build systems that aim at replacing Make (for Ninja) and CMake (for Meson). Both fuse and sshfs rely on them.
```
$ sudo apt install meson
```
will install both.

>Anecdote from their FAQ: *Why is it called Meson?
>When the name was originally chosen, there were two main limitations: there must not exist either a Debian package or a Sourceforge project of the given name. This ruled out tens of potential project names. At some point the name Gluon was considered. Gluons are elementary particles that hold protons and neutrons together, much like a build system's job is to take pieces of source code and a compiler and bind them to a complete whole.
Unfortunately this name was taken, too. Then the rest of subatomic particles were examined and Meson was found to be available.*

# 1. Install Fuse
Download and untar the latest release of [Fuse](https://github.com/libfuse/libfuse/releases). In this example, it is `fuse-3.9.3`:

```bash
$ cd fuse-3.9.3/
$ mdkir build; cd build
$ meson ..
$ ninja
$ sudo ninja install
```

# 2. Install SSHFS
Download and untar the latest release of [SSHFS](https://github.com/libfuse/sshfs/releases). In this example, it is `sshfs-3.7.0`.

```bash
$ cd sshfs-3.7.0/
$ mkdir build; cd build
$ meson ..
$ ninja
$ sudo ninja install
```

Now try running `sshfs`. A successful installation will look like:
```
$ sshfs
missing host
see `sshfs -h' for usage
```
> If it crashes like this:
>```
>$ sshfs                                                 
>sshfs: error while loading shared libraries: libfuse3.so.3: cannot open shared object file: No such file or directory
>```
>You might have to add a symbolic link manually or `sshfs` would fail to find `libfuse3.so.3` after installation.
>```
>$ sudo ln -s /usr/local/lib/x86_64-linux-gnu/libfuse3.so.3.9.3 /lib/x86_64-linux-gnu/libfuse3.so.3
>```
{.is-warning}





# 3. Mount your `$HOME`

Now you can mount your SDF home directory like this:
```
$ sudo mkdir /sdf
$ sudo chown $USER /sdf
$ sshfs YOUR_USER@sdf-login.slac.stanford.edu:/sdf/home/YOUR_HOME /sdf
```
where `YOUR_USER` is your login username, and you need to figure out what is `YOUR_HOME` in SDF. A way to find out is to log onto [OnDemand](https://ondemand-dev.slac.stanford.edu), click on the tab `Clusters > SDF Shell Access`, enter your password and type `pwd`.

Now you can for example put your code in your SDF home, and edit from your computer by opening `/sdf/your_code` in your favorite code editor. Enjoy! (and please report if there are inaccuracies in this page)
