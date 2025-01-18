

# Compiling Linux Kernel and Creating QEMU Images Using Docker



## *1. Overview*

In this project, we will:
1. Set up a Docker environment to compile the Linux kernel.
2. Create a QEMU disk image.
3. Boot the compiled kernel using QEMU for testing.

This guide assumes you’re using **Windows with WSL (Ubuntu)** and have no prior experience with Docker, QEMU, or kernel compilation.


## *2. Prerequisites*

1. **Windows Subsystem for Linux (WSL)**:
   - Install WSL and Ubuntu by following the official guide: [Install WSL](https://learn.microsoft.com/en-us/windows/wsl/install).

2. **Docker**:
   - Install Docker Desktop and enable WSL integration: [Install Docker](https://docs.docker.com/desktop/install/windows-install/).

3. **QEMU**:
   - Install QEMU inside WSL (Ubuntu):
     ```bash
     sudo apt update && sudo apt install qemu qemu-utils -y
     ```



## *3. Step-by-Step Guide*

### *Step 1: Set Up the Project Directory*

1. Open **WSL (Ubuntu)**.
2. Create a project directory:
   ```bash
   mkdir ~/kernel-build
   cd ~/kernel-build
   ```


### *Step 2: Download the Linux Kernel Source Code*

1. Download the Linux kernel source code (version 6.5):
   ```bash
   wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.xz
   ```

2. Extract the kernel source code:
   ```bash
   tar -xf linux-6.5.tar.xz
   cd linux-6.5
   ```



### *Step 3: Create a Dockerfile*

1. Create a `Dockerfile` in the `~/kernel-build/linux-6.5` directory:
   ```bash
   nano Dockerfile
   ```

2. Add the following content to the `Dockerfile`:
   ```Dockerfile
   FROM ubuntu:22.04
   RUN apt-get update && apt-get install -y \
       build-essential \
       libncurses-dev \
       bc \
       bison \
       flex \
       libssl-dev \
       qemu-utils \
       git \
       libelf-dev \
       cpio \
       qemu-system-x86
   WORKDIR /linux
   COPY . .
   ```

3. Save and exit the editor (`Ctrl + X`, then `Y` to confirm).



### *Step 4: Build the Docker Image*

1. Build the Docker image:
   ```bash
   docker build -t kernel-builder .
   ```



### *Step 5: Compile the Linux Kernel*

1. Start the Docker container:
   ```bash
   docker run -it --rm -v $(pwd):/linux kernel-builder
   ```

2. Configure the kernel:
   ```bash
   make defconfig
   ```

3. Compile the kernel:
   ```bash
   make -j$(nproc)
   ```

4. Verify the compiled kernel:
   ```bash
   ls -lh arch/x86/boot/bzImage
   ```



### *Step 6: Create a QEMU Disk Image*

1. Create a QEMU disk image:
   ```bash
   qemu-img create -f qcow2 myos.qcow2 10G
   ```



### *Step 7: Prepare the `initramfs`*

1. Create the `initramfs` directory:
   ```bash
   mkdir initramfs
   cd initramfs
   ```

2. Create the `init` script:
   ```bash
   echo -e '#!/bin/sh\n/bin/sh' > init
   chmod +x init
   ```

3. Package the `initramfs`:
   ```bash
   find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
   ```

4. Return to the kernel source directory:
   ```bash
   cd ..
   ```



### *Step 8: Boot the Kernel with QEMU*

1. Run QEMU:
   ```bash
   qemu-system-x86_64 \
       -kernel arch/x86/boot/bzImage \
       -initrd initramfs.cpio.gz \
       -hda myos.qcow2 \
       -append "root=/dev/sda console=ttyS0" \
       -nographic
   ```

2. Exit QEMU:
   - Press **`Ctrl+A`**, then press **`X`**.



## *4. Clean Up*

1. Stop WSL (if needed):
   ```powershell
   wsl --shutdown
   ```

2. Delete the project directory (optional):
   ```bash
   rm -rf ~/kernel-build
   ```



## *5. Troubleshooting*

- *Missing Dependencies*: Install missing packages using `apt-get install`.
- *QEMU Not Found*: Ensure `qemu-system-x86` is installed.
- *Kernel Panic*: Verify the `init` script in the `initramfs`.



*"Have you ever wondered how your computer actually works? This project is your chance to dive into the heart of operating systems by building and testing your own Linux kernel! Using Docker and QEMU, you’ll learn how to compile the Linux kernel from scratch and run it in a virtual machine. Whether you’re a student, a hobbyist, or an aspiring developer, this hands-on guide will help you understand how operating systems work, teach you valuable skills like virtualization and containerization, and give you the confidence to tackle real-world projects like customizing Linux for IoT devices or testing software in isolated environments. No prior experience needed—just curiosity and a willingness to learn!"*

