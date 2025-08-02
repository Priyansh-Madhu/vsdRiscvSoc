# 🛠️ RISC-V Toolchain Setup on Rocky Linux (Task-wise with My Changes)

This repository documents my personal step-by-step setup of the RISC-V GCC Toolchain, Spike, Proxy Kernel, and simulation tools on **Rocky Linux 8**, as part of the VSD RISC-V Workshop.

## 🖥️ My System Configuration

    OS: Rocky Linux 8.9 (Green Obsidian) – x86_64

    Kernel: Linux 4.18.0-513.el8.x86_64

    Shell: bash 4.4.20

    CPU: Intel® Core™ i5-8250U CPU @ 1.60GHz × 4

    Memory: 8 GB RAM

    Disk: 100+ GB free

    User: priyansh

    Hostname: rocky8

## ✅ Task 1 – Install Base Developer Tools

Why: These are common build prerequisites (compilers, linkers, autotools) and libraries
required by the RISC‑V simulator, proxy kernel, and other tooling. GTKWaves is included for
waveform viewing in digital design flows.

📄 **From PDF (for Ubuntu):**

~~~bash
sudo apt-get install -y git vim autoconf automake autotools-dev curl \
libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex \
texinfo gperf libtool patchutils bc zlib1g-dev libexpat1-dev gtkwave
~~~

🛠️ **What I Did (Rocky Linux):**

~~~bash
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y git vim autoconf automake curl libmpc-devel \
mpfr-devel gmp-devel gawk bison flex texinfo gperf libtool patchutils \
bc zlib-devel libexpat-devel dtc ncurses-compat-libs gtkwave
~~~

**Why** Rocky Linux uses dnf instead of apt-get

## ✅ Task 2 – Create a Clean Workspace

Why: Keeps everything contained in ~/riscv_toolchain, making it easy to remove, update, or
archive. Storing $pwd (your home) ensures consistent paths for later steps.

📄 **From PDF:**

~~~bash
cd ~
pwd=$PWD
mkdir -p riscv_toolchain
cd riscv_toolchain
~~~
## ✅ Task 3 – Get Prebuilt RISC‑V GCC Toolchain

Why: Provides riscv64-unknown-elf-gcc (newlib) to compile bare‑metal/user‑space RISC‑V
programs. Using a prebuilt toolchain avoids a long source build.

📄 **From PDF:**

~~~bash
wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
tar -xvzf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-ubuntu14.tar.gz
~~~
## ✅ Task 4 – Add Toolchain to PATH

Why: Allows riscv64-unknown-elf-gcc and related binaries to be available without typing
full paths, both now and after you reopen the terminal.

**Command (current shell):**

~~~bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/bin:$PATH
~~~

**Command (persistent for new terminals):**

~~~bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
~~~

## ✅ Task 5 — Install Device Tree Compiler (DTC)

Why: Some RISC‑V components expect DTC to be present; it’s a common dependency in
SoC/simulator flows.

📄 **From PDF:**

~~~bash
sudo apt-get install -y device-tree-compiler
~~~

🛠️ **I replaced this with the Rocky-specific equivalent:**

~~~bash
sudo dnf install -y dtc
~~~

## ✅ Task 6 – Build & Install Spike (RISC-V ISA Simulator)

Why: Spike is the reference ISA simulator. You’ll run your compiled RISC‑V ELF on Spike to
validate correctness. Installing into the same prefix keeps everything together.

📄 **From PDF:**

~~~bash
git clone https://github.com/riscv/riscv-isa-sim.git
cd riscv-isa-sim
mkdir build && cd build
../configure --prefix=...
make -j$(nproc)
sudo make install
~~~

## ✅ Task 7 – Build and install the RISC‑V Proxy Kernell (RISC-V pk)

Why: pk provides a minimal runtime so you can run newlib ELF binaries under Spike with
'spike pk ./your_prog'. It bridges your compiled program to the simulator.

📄 **From PDF:**

~~~bash
git clone https://github.com/riscv/riscv-pk.git
cd riscv-pk
mkdir build && cd build
../configure --prefix=... --host=riscv64-unknown-elf
make -j$(nproc)
sudo make install
~~~

## ✅ Task 8 — Ensure the cross bin directory is in PATH

Why: Some installs place pk and related utilities into a nested riscv64-unknown-elf/bin.
Adding this ensures pk is found by 'which pk'.

**Command (current shell):**

~~~bash
export PATH=$pwd/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH
~~~

**Command (persistent):**

~~~bash
echo 'export PATH=$HOME/riscv_toolchain/riscv64-unknown-elf-gcc-8.3.0-
2019.08.0-x86_64-linux-ubuntu14/riscv64-unknown-elf/bin:$PATH' >>
~/.bashrc
source ~/.bashrc
~~~

## ✅ Task 10 — Quick sanity checks

Why: Confirms the toolchain and simulator are visible and runnable from your shell.

📄 **From PDF:**

~~~bash
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
which spike
which pk
~~~

## ✅ Final Deliverable – Unique C Test

**1) Create unique_test.c**

~~~bash
#include <stdint.h>
#include <stdio.h>

#ifndef USERNAME
#define USERNAME "unknown_user"
#endif

#ifndef HOSTNAME
#define HOSTNAME "unknown_host"
#endif

static uint64_t fnv1a64(const char *s) {
    const uint64_t FNV_OFFSET = 1469598103934665603ULL;
    const uint64_t FNV_PRIME  = 1099511628211ULL;
    uint64_t h = FNV_OFFSET;
    for (const unsigned char *p = (const unsigned char*)s; *p; ++p) {
        h ^= (uint64_t)(*p);
        h *= FNV_PRIME;
    }
    return h;
}

int main(void) {
    const char *user = USERNAME;
    const char *host = HOSTNAME;
    char buf[256];
    int n = snprintf(buf, sizeof(buf), "%s@%s", user, host);
    if (n <= 0) return 1;
    uint64_t uid = fnv1a64(buf);
    printf("RISC-V Uniqueness Check\\n");
    printf("User: %s\\n", user);
    printf("Host: %s\\n", host);
    printf("UniqueID: 0x%016llx\\n", (unsigned long long)uid);
#ifdef __VERSION__
    printf("GCC_VLEN: %llu\\n", (unsigned long long)(sizeof(__VERSION__) - 1));
#endif
    return 0;
}
~~~

**2) Compile with injected identity and RISC‑V flags**

~~~bash
riscv64-unknown-elf-gcc -O2 -Wall -march=rv64imac -mabi=lp64 \
-DUSERNAME="$(id -un)" -DHOSTNAME="$(hostname -s)" \
unique_test.c -o unique_test
~~~

**3) Run on Spike with the proxy kernel**

~~~bash
spike pk ./unique_test
~~~

**Output**

~~~bash
RISC-V Uniqueness Check
User: priyansh
Host: rocky8
UniqueID: 0x4fd71a1ee6bc2374
GCC_VLEN: 12
~~~
