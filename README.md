# MesaLock Linux

MesaLock Linux is a general purpose Linux distribution which aims to provide a
*safe* and *secure* user space environment. To eliminate high-severe memory
corruption vulnerabilities, all user space applications are rewritten in
*memory safe* programming languages like Rust and Go.  This extremely reduces
attack surfaces of an operating system exposed in the wild, leaving the
remaining attack surfaces auditable and convergent. In addition to having a
broad hardware compatibility provided by the Linux kernel, MesaLock Linux can
also substantially improve the security of the Linux ecosystem. The current two
main usage scenarios of MesaLock Linux are docker image and security sensitive
embedded devices. With the development and maturity of the ecosystem, MesaLock
Linux can be also adopted in the server environment.

## Building

Currently, MesaLock Linux is provided in two versions: live ISO and rootfs. The
live ISO image can be used to create a bootable live USB, or boot in a virtual
machine. The rootfs (i.e., root file system) can be used as a minimal root
image for a container.

### Requirements

#### Clone MesaLock repository

Clone `mesalock-distro` repositry recursively with its submodules (packages).

```sh
$ git clone --recursive http://172.19.62.4/mesalock-linux/mesalock-distro.git
$ cd mesalock-distro
```

#### Build in Docker

We provide a `Dockerfile` for building MesaLock Linux with all dependencies
installed. Please build the docker image first and then in the building container
environment, you can build packages, live ISO, and rootfs.

```sh
$ docker build -t mesalock-linux/build-mesalock-linux --rm build-dockerfile
$ docker run -v $(pwd):/mesalock-distro -w /mesalock-distro \
    -it mesalock-linux/build-mesalock-linux /bin/bash
```

#### Build on Ubuntu

You can also build a Ubuntu machine, please install these building dependencies
first:

```sh
$ # install packages
$ apt-get update && \
  apt-get install -q -y --no-install-recommends \
           curl \
           git \
           build-essential \
           wget \
           bc \
           gawk \
           parallel \
           pigz \
           cpio \
           xorriso \
           fakeroot \
           syslinux-utils \
           uuid-dev \
           libmpc-dev \
           libisl-dev \
           libz-dev \
           software-properties-common

$ # install Go
$ add-apt-repository -y ppa:gophers/archive && \
  apt-get update && \
  apt-get install -q -y --no-install-recommends \
           golang-1.9-go

$ # install Rust
$ curl https://sh.rustup.rs -sSf | sh -s -- -y && \
    rustup default nightly

$ # setup PATH
$ export PATH="$HOME/.cargo/bin:/usr/lib/go-1.9/bin:$PATH"
```

### Build packages, live ISO, and rootfs

After installing building dependencies, you can run following commands to build
packages, live ISO, and rootfs.

  - First build all packages: `./mkpkg`
  - Build the live ISO: `./mesalockiso`
  - Build the container rootfs: `./mesalockrootfs`
  - Build a specific package only: `./mkpkg <package_name>`

The live ISO (`mesalock-linux.iso`) and rootfs (`rootfs.tar.gz`) can be found
in the `build` directory.

## Trying

MesaLock Linux can be run in real devices (e.g., boot from a Live USB), virtual
machines, and docker containers.

### Virtual machine

You can try MesaLock Linux with Live ISO or in a docker container. Here are
steps to try MesaLock Linux in VirtualBox.

  1. Open VirtualBox and "New" a VM.
  2. In the VM settings, choose `mesalock-linux.iso` as "Optical Drive".
  3. Start the VM and explore MesaLock Linux.

### Docker container

We provide a simple `Dockerfile` for MesaLock Linux. Here are steps to try
MesaLock Linux in a docker container.

  1. Copy rootfs into the docker directory: `cp build/rootfs.tar.gz mesalockrootfs-dockerfile/`
  2. Build the docker image: `docker build --rm -t mesalock-linux/mesalock-linux mesalockrootfs-dockerfile`
  3. Run the image and expeience MesaLock Linux: `docker run --rm -it mesalock-linux/mesalock-linux`

### Example: hosting web servers

The `mesalock-demo` package provides several examples and will be installed
under the `/root/mesalock-demo` directory. For instance, we made several web
server demos written in [Rocket](https://github.com/SergioBenitez/Rocket/),
which is a web framework written in Rust.  To try these demos in the VM, please
follow these instructions.

  1. In the VM settings, select "NAT" for network adapter and use port
     forwarding function in the advanced settings to bind host and guest
     machines. Here we add a new rule to bind host IP (127.0.0.1:8080) with
     guest IP (10.0.2.15:8000).
  2. Start MesaLock Linux.
  3. Bring up all network devices. Here we use `ip` command:
    ```
    $ ip link set lo up
    $ ip link set eth0 up
    ```
  4. Setup IP address of the network devices.
    ```
    $ ip address add 10.0.2.15/24 dev eth0
    ```
  5. Run a web server.
    ```
    $ cd /root/mesalock-demo/rocket-hello-world && ./hello_world
    $ # or
    $ cd /root/mesalock-demo/rocket-tls && ./tls
    ```
  6. Finally, connect to the web server using a browser. In this example, type
     in http://127.0.0.1:8080 in the browser.

## Packages

MesaLock Linux provides many packages with memory safety in mind. All user
space applications are written in Rust and Go. Thanks to the open source
community, they have crated many useful and high-quality tools. The number of
packages will increase as the time goes on.

  - `brotli`: compression tool written in Rust ([dropbox/rust-brotli](https://github.com/dropbox/rust-brotli))
  - `busybox`: busybox tool set for testing only ([busybox](https://busybox.net))
  - `exa`: replacement for ls written in Rust ([ogham/exa](https://the.exa.website))
  - `fd-find`: simple, fast and user-friendly alternative to find ([sharkdp/fd](https://github.com/sharkdp/fd))
  - `filesystem`: base filesystem layout (maintained by MesaLock Linux)
  - `gcc-libs`: GCC library, only libgcc_s.so is used ([gcc](https://gcc.gnu.org/))
  - `giproute2`: ip tool written in Go (maintained by MesaLock Linux)
  - `glibc`: the GNU C library ([glibc](https://www.gnu.org/software/libc/))
  - `init`: init script (maintained by MesaLock Linux)
  - `ion-shell`: shell written in Rust ([redox-os/ion](https://github.com/redox-os/ion))
  - `linux`: Linux kernel ([linux](https://www.kernel.org/))
  - `mesalock-demo`: some demo projects (maintained by MesaLock Linux)
  - `mgetty`: getty written in Rust (maintained by MesaLock Linux)
  - `micro`: modern and intuitive terminal-based text editor in written Go ([zyedidia/micro](https://github.com/zyedidia/micro))
  - `minit`: init written in Rust (maintained by MesaLock Linux)
  - `ripgrep`: ripgrep combines the usability of The Silver Searcher with the raw
    speed of grep, written in Rust ([BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep))
  - `syslinux`: bootloader ([syslinux](http://www.syslinux.org/wiki/index.php?title=The_Syslinux_Project))
  - `tokei`: count your code, quickly, in Rust ([Aaronepower/tokei](https://github.com/Aaronepower/tokei))
  - `tzdata`: timezone data ([tzdata](https://www.iana.org/time-zones))
  - `uutils-coreutils`: cross-platform Rust rewrite of the GNU coreutils ([uutils/coreutils](https://github.com/uutils/coreutils))
  - `uutils-findutils`: rust implementation of findutils ([uutils/findutils](https://github.com/uutils/findutils))
  - `xi-core`: a modern editor with a backend written in Rust ([google/xi-editor](https://github.com/google/xi-editor))
  - `xi-tui`: a tui frontend for Xi ([little-dude/xi-tui](https://github.com/little-dude/xi-tui))

## Contributing

MesaLock Linux is very young and at an early stage. Some important components
are still missing. Building a safe and secure Linux distro relies on the whole
community. You are very welcome to contribute to the MesaLock Linux project.

You can get involved in various forms:

  - Improving MesaLock Linux: building process, integration issues, etc.
  - Contributing to core packages of MesaLock Linux: minit, mgetty, etc.
  - Writing system tools with security in mind: tools written in memory safe
    language such as Rust and Go.

## Maintainer

  - Mingshen Sun `<sunmingshen@baidu.com>`

## License

MesaLock Linux is provided under the [BSD license](LICENSE).
