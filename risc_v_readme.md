# 🛠️ Task 1: RISC-V Toolchain Setup Tasks & Uniqueness Test

[![RISC-V](https://img.shields.io/badge/Architecture-RISC--V-blue.svg)](https://riscv.org/)
[![Toolchain](https://img.shields.io/badge/Toolchain-RISC--V%20GCC-blueviolet.svg)]()
[![Rocky Linux](https://img.shields.io/badge/Platform-Rocky%20Linux%208.3-10B981.svg)](https://rockylinux.org/)
[![Status](https://img.shields.io/badge/Status-✅%20Complete-success.svg)]()

A comprehensive guide for setting up a complete RISC-V development toolchain on Rocky Linux 8.3 with detailed troubleshooting and solutions for common issues.

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Installation Steps](#-installation-steps)
- [Uniqueness Test](#-uniqueness-test)
- [Issues Encountered & Solutions](#-issues-encountered--solutions)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project documents the complete setup process for a RISC-V development toolchain on Rocky Linux 8.3, including installation procedures and a unique identification test program.

**Content:**
- Installation of RISC-V toolchain components
- Configuration and path setup
- Custom uniqueness test implementation
- Comprehensive troubleshooting guide

## 🔧 Prerequisites

### System Requirements
- **OS**: Rocky Linux 8.3 (64-bit)  
- **RAM**: 4-8 GB recommended
- **Storage**: 30 GB free space
- **CPU**: 2+ cores recommended
- **Access**: Root privileges required

## 🚀 Installation Steps

### Step 1: Installing Base Developer Tools

These are basic packages ("dependencies") you'll need for compiling code and building RISC-V tools.

We are using the Rocky Linux 8.3 operating system rather than Ubuntu 22.04 LTS (64-bit) and made changes in the script according to that as it requires using dnf (YUM-based) commands instead of apt.

Open the terminal by default it will be at default location in your pc with account name you have saved. Type the command `su -- root` and add type the correct password to access root.because you will need sudo permissions to install the dependencies.

Run:

```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y epel-release
sudo dnf install -y git vim autoconf automake make gcc gcc-c++ \
mpfr-devel gmp-devel libmpc-devel gawk bison flex texinfo \
gperf libtool patchutils bc zlib-devel expat-devel wget curl
```

**Additional notes:**

gtkwave is optional and may not be in base repos. If needed:

```bash
sudo dnf install -y gtkwave
```

<img width="1366" height="768" alt="step 1" src="https://github.com/user-attachments/assets/1188366d-9f14-4c73-9a75-44d6344e018e" />


### Step 2: Create a clean workspace and capture your home path

Make directory named riscv_toolchain and open it. It keeps your files organized in one place.

```bash
cd ~
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
```
<img width="1366" height="768" alt="step2" src="https://github.com/user-attachments/assets/904da785-086f-4735-aa08-baebb30c3d3a" />


### Step 3: Download and Extract Prebuilt RISC-V GCC Toolchain

This provides the special RISC-V C compiler ("riscv64-unknown-elf-gcc").

```bash
wget "https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz"
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
```

### Step 4: Add the Toolchain to Your PATH

This lets you use "riscv64-unknown-elf-gcc" without typing its full location.

- **For your current terminal session:**

```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
```

- For all future terminals (persistent):

```bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Step 5: Install Device Tree Compiler (DTC)

Another needed piece for running the simulator.

We got error while installing Device tree compiler because **EPEL** and **PowerTools /CodeReady Builder** are required to unlock all the developer tools (like texinfo, gperf) on Rocky Linux.Without enabling these, commands like sudo dnf install texinfo gperf might fail.

These all are supporting steps for installing/compiling the Device Tree Compiler (DTC), which is needed for RISC-V simulation and development.

```bash
sudo dnf install -y epel-release
sudo dnf update -y
sudo dnf install -y texinfo gperf
dnf search texinfo
dnf search gperf
sudo dnf config-manager --set-enabled powertools
sudo dnf config-manager --set-enabled codeready-builder-for-rhel-8-x86_64-rpms
sudo dnf update -y
sudo dnf install -y texinfo gperf
```

Then we gave this command to install dtc and successfully installed it.

```bash
sudo dnf install -y dtc
```

### Step 6: Build and Install Spike (RISC-V Simulator)

Spike runs your compiled RISC-V programs.

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
make -j$(nproc)
sudo make install
```

We got some error about gcc path not found properly and some other errors. Then we checked if the gcc was installed in our PC or not, we found that it was already installed but there was some issues regarding version and path detection.

Then gave these commands in the terminal and resolved the error:

```bash
# Check if spike binary exists
ls -la /root/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/spike

# Test spike
/root/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/bin/spike --help

# Set environment variables
export RISCV=/root/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14
export PATH=$RISCV/bin:$PATH

# Add to bashrc for permanent setup
echo 'export RISCV=/root/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14' >> ~/.bashrc
echo 'export PATH=$RISCV/bin:$PATH' >> ~/.bashrc

# Source the changes
source ~/.bashrc

# Test RISC-V GCC
riscv64-unknown-elf-gcc --version

# Test Spike simulator
spike --help

# Test other RISC-V tools
riscv64-unknown-elf-objdump --version
```

### Step 7: Build and install the RISC‑V Proxy Kernel (riscv-pk)

We got some errors in this part of the installation while cloning to the git hub link.

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 \
--host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```

So I made some change the original code and used this code:

```bash
cd $pwd/riscv_toolchain
rm -rf riscv-pk  # Remove problematic version
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
git checkout v1.0.0
mkdir -p build && cd build
../configure --prefix=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14 --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
```

### Step 8: Ensure Proxy Kernel Utilities Are in Your PATH

Some installs place pk and related utilities into a nested riscv64-unknown-elf/bin.

Adding this ensures pk is found by 'which pk'.

```bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Step 9: (Optional) Install Icarus Verilog

Code used:

```bash
cd $pwd/riscv_toolchain
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
git checkout --track -b v10-branch origin/v10-branch
chmod +x autoconf.sh
./autoconf.sh
./configure
make -j$(nproc)
sudo make install
```

![verilog pic](https://github.com/user-attachments/assets/6dc39887-e5b6-4ea8-9d7f-8960fa554cb6)

### Step 10: Quick Sanity Checks

Test if the main tools are working:

```bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
spike --version
which pk
```

- **For gcc location confirmation**

![gcc confirmation](https://github.com/user-attachments/assets/130975ff-515a-4564-bc26-0a7bb78cf9e8)


- **For spike confirmation**

Got error in **spike --version** as it was not accepting this command so used this command **git log --l** to get version details

![spike confirmation](https://github.com/user-attachments/assets/1f9ae4f0-1e64-4561-b72d-899af26d8f35)

- **For proxy kernel confirmation**
  ![pk confirmation](https://github.com/user-attachments/assets/83a53f30-8d9e-423d-a8be-7457620fe6ab)

## 🧪 Uniqueness Test

### Step 11: A Unique C Test (Username & Machine Dependent)

Created the Unique test C file:

```bash
#include <stdint.h>
#include <stdio.h>

#ifndef PALAK_PARMAR
#define PALAK_PARMAR "PALAK_PARMAR"
#endif

#ifndef PDEU
#define PDEU "PDEU"
#endif

// 64-bit FNV-1a hash function
static uint64_t fnv1a64(const char *s) {
    const uint64_t FNV_OFFSET = 1469598103934665603ULL;
    const uint64_t FNV_PRIME = 1099511628211ULL;
    uint64_t h = FNV_OFFSET;
    for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
        h ^= (uint64_t)(*p);
        h *= FNV_PRIME;
    }
    return h;
}

int main(void) {
    const char *user = PALAK_PARMAR;
    const char *host = PDEU;
    char buf[256];
    int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
    if (n <= 0) return 1;
    uint64_t uid = fnv1a64(buf);
    printf("RISC-V Uniqueness Check\n");
    printf("User: %s\n", user);
    printf("Host: %s\n", host);
    printf("UniqueID: 0x%016llx\n", (unsigned long long)uid);
    return 0;
}
```

I compiled the code using this command:

```bash
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME="$(id -un)" -DHOSTNAME="$(hostname -s)" \
unique_test.c -o unique_test
```

Run the program in spike:

```bash
spike pk ./unique_test
```

<img width="920" height="360" alt="Unique Test Execution" src="https://github.com/user-attachments/assets/unique-test-execution.png" />

### 🎯 Unique ID Results

```bash
User: PALAK_PARMAR
Host: PDEU
UniqueID: 0x3c505c397f1fa2b6
GCC_VLEN: 5
```

## 🚨 Issues Encountered & Solutions

### 1. Configure Script: "C Compiler Cannot Create Executables"

- **Symptom:**  
  During build (e.g., Spike/Proxy Kernel), the configure script failed:

```
C compiler cannot create executables
```

- **Cause:**  
  Missing development packages (e.g., `gcc`, `glibc-devel`, etc.)

- **Solution:**  
  Install full development environment:

```bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y gcc gcc-c++ glibc-devel ...
```

### 2. Native Build Fails: "as: unrecognized option --64"

- **Symptom:**  
  Compiling native C code or proxy kernel gave this assembler error:  
  `as: unrecognized option '--64'`

- **Cause:**  
  RISC-V toolchain's `bin` directory was at the start of your `$PATH`, causing the native GCC to invoke the RISC-V assembler instead of the system's x86_64 assembler.

- **Solution:**
  - Remove any permanent RISC-V `$PATH` entries from your `~/.bashrc` or `~/.bash_profile`.
  - **Use per-session environment scripts**:
    - `~/set_riscv_env.sh` --- activates the RISC-V toolchain when needed
    - `~/set_native_env.sh` --- restores default native Linux environment
  - Only activate RISC-V environment when cross-compiling RISC-V code.

### 3. Proxy Kernel Build: "gcc: error: unrecognized argument in option '-mcmodel=medany'"

- **Symptom:**  
  Make failed with:  
  `unrecognized argument in option '-mcmodel=medany'`

- **Cause:**  
  Proxy kernel was being built with native GCC instead of RISC-V cross-compiler.

- **Solution:**

Activate RISC-V environment first:

```bash
source ~/set_riscv_env.sh
```

Then configure/build again.

### 4. Spike Does Not Support `--version`

- **Symptom:**  
  `spike --version` error: "unrecognized option --version"

- **Cause:**  
  Some Spike builds from source do not support the `--version` flag.

- **Solution:**  
  Use `spike -h`, or check the Git commit in the source directory:

```bash
cd ~/riscv_toolchain/riscv-isa-sim
git log -1
```

to document the build provenance.

### 5. Installing Packages: "command not found"

- **Symptom:**  
  Running `mpfr-devel gmp-devel ...` gave "command not found."

- **Cause:**  
  Package names must be installed via `dnf install ...`, not run directly.

- **Solution:**

```bash
sudo dnf install -y mpfr-devel gmp-devel ...
```

## 🔧 Permanent Solution Adopted

- **Never** pollute global `$PATH` with RISC-V toolchain by default.
- Switch environment using scripts only as needed.
- Ensure cross-compiler is only active during RISC-V builds, keeping the native development safe and functional.

## 📝 Summary

Most issues came from `$PATH` conflicts between native and RISC-V toolchains. Clean per-session environment management and verifying which compiler is being used solved all problems. Following these solutions ensures reliable future usage and avoids repeat errors.

## 📁 Project Structure

```
riscv_toolchain/
├── riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14/
│   ├── bin/                    
│   ├── lib/                    
│   └── riscv64-unknown-elf/   
├── riscv-isa-sim/              
├── riscv-pk/                   
├── iverilog/                  
├── unique_test.c               
└── unique_test                
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
