# Build and install CRI-O from source

This guide will walk you through the installation of [CRI-O](https://github.com/cri-o/cri-o), an Open Container Initiative-based implementation of [Kubernetes Container Runtime Interface](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/container-runtime-interface-v1.md).

It is assumed you are running a Linux machine.

## Runtime dependencies

- runc, Clear Containers runtime, or any other OCI compatible runtime
- socat
- iproute
- iptables

Latest version of `runc` is expected to be installed on the system. It is picked up as the default runtime by CRI-O.

## Build and Run Dependencies

**Required**

Fedora, CentOS, RHEL, and related distributions:

```bash
yum install -y \
  btrfs-progs-devel \
  containers-common \
  device-mapper-devel \
  git \
  glib2-devel \
  glibc-devel \
  glibc-static \
  go \
  golang-github-cpuguy83-go-md2man \
  gpgme-devel \
  libassuan-devel \
  libgpg-error-devel \
  libseccomp-devel \
  libselinux-devel \
  ostree-devel \
  pkgconfig \
  runc
```

Debian, Ubuntu, and related distributions:

```bash
# Add containers-common and cri-o-runc
apt-add-repository ppa:projectatomic/ppa

apt-get install -y \
  btrfs-tools \
  containers-common \
  git \
  golang-go \
  libassuan-dev \
  libdevmapper-dev \
  libglib2.0-dev \
  libc6-dev \
  libgpgme11-dev \
  libgpg-error-dev \
  libseccomp-dev \
  libsystemd-dev \
  libselinux1-dev \
  pkg-config \
  go-md2man \
  cri-o-runc
```

**Caveats and Notes:**

Debian, Ubuntu, and related distributions will also need a copy of the development libraries for `ostree`, either in the form of the `libostree-dev` package from the [flatpak](https://launchpad.net/~alexlarsson/+archive/ubuntu/flatpak) PPA, or built [from source](https://github.com/ostreedev/ostree) (more on that [here](https://ostree.readthedocs.io/en/latest/#building)).

If using an older release or a long-term support release, be careful to double-check that the version of `runc` is new enough (running `runc --version` should produce `spec: 1.0.0`), or else build your own.

Be careful to double-check that the version of golang is new enough, version 1.8.x or higher is required.  If needed, golang kits are available at https://golang.org/dl/

## Get Source Code

Clone the source code using:

```bash
git clone https://github.com/cri-o/cri-o # or your fork
cd cri-o
```

Make sure your `CRI-O` and `kubernetes` versions are of matching major versions. For instance, if you want to be compatible with the latest kubernetes release, you'll need to use the latest tagged release of `CRI-O` on branch release-1.14

## Build

To install with default buildtags using seccomp, use:

```bash
make
sudo make install
```

Otherwise, if you do not want to build `CRI-O` with seccomp support you can add `BUILDTAGS=""` when running make.

```bash
make BUILDTAGS=""
sudo make install
```

### Build Tags

`CRI-O` supports optional build tags for compiling support of various features.
To add build tags to the make option the `BUILDTAGS` variable must be set.

```bash
make BUILDTAGS='seccomp apparmor'
```

| Build Tag | Feature                            | Dependency  |
|-----------|------------------------------------|-------------|
| seccomp   | syscall filtering                  | libseccomp  |
| selinux   | selinux process and mount labeling | libselinux  |
| apparmor  | apparmor profile support           | <none>      |

## Setup CNI networking

A proper description of setting up CNI networking is given in the
[`contrib/cni` README](/contrib/cni/README.md). But the gist is that you need to
have some basic network configurations enabled and CNI plugins installed on
your system.

## CRI-O configuration

If you are installing for the first time, generate and install configuration files with:

```
sudo make install.config
```

### Validate registries in registries.conf

Edit `/etc/containers/registries.conf` and verify that the registries option has valid values in it.  For example:

```
[registries.search]
registries = ['registry.access.redhat.com', 'registry.fedoraproject.org', 'quay.io', 'docker.io']

[registries.insecure]
registries = []

[registries.block]
registries = []
```

For more information about this file see [registries.conf(5)](https://github.com/containers/image/blob/master/docs/containers-registries.conf.5.md).

### Recommended - Use systemd cgroups.

By default, CRI-O uses cgroupfs as a cgroup driver. However, we recommend using systemd as a cgroup driver. You can change your cgroup driver in crio.conf:

```
cgroup_driver = "systemd"
```

### Optional - Modify verbosity of logs in /etc/crio/crio.conf

Users can modify the `log_level` field in `/etc/crio/crio.conf` to change the verbosity of
the logs.
Options are fatal, panic, error (default), warn, info, and debug.

```
log_level = "info"
```

### Optional - Modify capabilities and sysctls
By default, `CRI-O` uses the following capabilities:

```
default_capabilities = [
	"CHOWN",
	"DAC_OVERRIDE",
	"FSETID",
	"FOWNER",
	"NET_RAW",
	"SETGID",
	"SETUID",
	"SETPCAP",
	"NET_BIND_SERVICE",
	"SYS_CHROOT",
	"KILL",
]
```
and no sysctls
```
default_sysctls = [
]
```

Users can change either default by editing `/etc/crio/crio.conf`.

## Starting CRI-O

Running make install will download CRI-O into the folder
```bash
/usr/local/bin/crio
```

You can run it manually there, or you can set up a systemd unit file with:

```
sudo make install.systemd
```

And let systemd take care of running CRI-O:

``` bash
sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
```

## Using CRI-O

- Follow this [tutorial](crictl.md) to quickly get started running simple pods and containers.
- To run quickly with kubernetes, see [our quick kuberentes guide](quick-kubernetes.md).
- To run a full cluster, see [the instructions](kubernetes.md).
- To run with kubeadm, see [kubeadm instructions](kubeadm.md).
