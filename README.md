# ft_linux - Building a Linux Distribution from Source

![Language](https://img.shields.io/badge/language-Bash%20%2F%20C-blue)
![Architecture](https://img.shields.io/badge/arch-x86__64-lightgrey)
![Type](https://img.shields.io/badge/type-Linux%20From%20Scratch-red)
![Build](https://img.shields.io/badge/build-manual-orange)

<br />
<p align="center">
  <h3 align="center">A fully functional Linux OS built without a package manager</h3>
</p>

## 🗣️ About The Project

**ft_linux** is an advanced systems engineering project based on the **Linux From Scratch (LFS)** documentation. The objective is to build a custom, minimal Linux distribution by compiling every single component from source code—from the Binutils and GCC compiler to the Bash shell and the Linux Kernel itself.

This project is the ultimate test of understanding **how an Operating System is put together**. It moves beyond "administering" a system to "architecting" one, requiring a deep understanding of dynamic linking, compiler toolchains, and the boot process.

### 🎯 Key Engineering Goals
- **Toolchain Construction:** Solving the "chicken and egg" problem by building a cross-compiler (Binutils, GCC, Glibc) to build the final system.
- **Dependency Hell Management:** Manually resolving shared library dependencies for core utilities without `apt`, `yum`, or `pacman`.
- **Kernel Compilation:** Configuring (`make menuconfig`) and building a monolithic Linux Kernel tailored specifically to the host hardware.
- **Bootloader Setup:** Manually installing and configuring GRUB to hand off control to the kernel image.

---

## 🏗️ Build Architecture

The construction process follows a strict 3-stage isolation model to prevent the host system's libraries from contaminating the new distribution.

~~~mermaid
graph LR
    A[Host System] -->|Builds| B(Temporary Toolchain)
    B -->|Builds inside Chroot| C(Final System)
    C -->|Bootable| D[ft_linux Distro]
~~~

1.  **Stage 1 - Cross-Toolchain:** Building a hardware-agnostic compiler and linker.
2.  **Stage 2 - Temporary Tools:** Building a minimal set of isolated tools (`/tools`) linked against the new Glibc.
3.  **Stage 3 - Final System:** Entering the `chroot` environment to build the permanent system binaries (`/bin`, `/usr/bin`) using the temporary tools.

---

## 🛠️ Technical Implementation Highlights

### 1. The Glibc & GCC Dance
The most critical part of the project was compiling the **GNU C Library (Glibc)**. Since every program in the system depends on it, it had to be compiled with absolute precision to ensure it linked correctly to the new dynamic loader (`ld-linux.so`), decoupling it completely from the host OS.

### 2. Kernel Configuration
Instead of using a generic distribution kernel (which includes drivers for every possible hardware), I stripped the kernel down to the bare essentials:
* **Filesystems:** Added support only for `ext4`.
* **Drivers:** Included only virtio/SATA drivers required for the disk.
* **Network:** Enabled TCP/IP stack but removed legacy protocols.
* **Result:** A kernel image significantly smaller and faster to boot than standard kernels.

### 3. Root Filesystem Structure
I manually created the Unix directory hierarchy standard (FHS):
* `/etc` for configuration (fstab, hosts, passwd).
* `/dev` populated via `mknod` or `devtmpfs` mounting.
* `/proc` and `/sys` virtual filesystems for kernel interaction.

---

## ✅ System Validation

The successful completion of the project is defined by the ability to boot the machine independently and perform basic system operations:

1.  **Boot Process:** The BIOS loads GRUB, which successfully loads the compiled `vmlinuz` kernel.
2.  **Init System:** The kernel mounts the root filesystem (`/`) and executes `/sbin/init`.
3.  **User Space:** A login prompt appears (`getty`), allowing entry as `root`.
4.  **Networking:** The network interface (`eth0`) is up and capable of connecting to the internet via ping/ssh.

---
*Built by Eleder Andres. "Where there is a shell, there is a way."*
