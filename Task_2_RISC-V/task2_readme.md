# 🛠️ Task 2: Prove Your Local RISCV Setup (Run, Disassemble,Decode)
[![RISC-V](https://img.shields.io/badge/Architecture-RISC--V-blue.svg)](https://riscv.org/)
[![Toolchain](https://img.shields.io/badge/Toolchain-RISC--V%20GCC-blueviolet.svg)]()
[![Rocky Linux](https://img.shields.io/badge/Platform-Rocky%20Linux%208.3-10B981.svg)](https://rockylinux.org/)
[![Status](https://img.shields.io/badge/Status-✅%20Complete-success.svg)]()

Task 2 involves proving your local RISC-V setup by running several RISC-V C programs on your machine using your locally compiled toolchain and the Spike simulator. 

## 🎯 Overview
This repository showcases the development and testing of four RISC-V C programs, compiled using a locally installed RISC-V toolchain and run on the Spike simulator with the proxy kernel (pk). 
Each program incorporates a custom identification feature that embeds the system’s username, hostname, machine ID, and timestamps, ensuring the generated outputs are uniquely tied to this specific machine.

##  🚀 Task Implementation

 ### 🔧 Spike Version
To find its version we use the following command :
```
git log --l
which spike
```
![spike confirmation](https://github.com/user-attachments/assets/99ddfae1-88d7-4a85-ac54-56485bb1aa4e)

### 🔧 GCC Toolchain Information
To find its version we use the following command :
```
which riscv64-unknown-elf-gcc
riscv64-unknown-elf-gcc -v
```
<img width="1366" height="768" alt="Set the correct PATH after redownloading the tool chain" src="https://github.com/user-attachments/assets/71af006b-8fb8-49fc-870d-fd9a91a4e076" />

 ### A. Uniqueness mechanism (do this before compiling)
Before compiling, set environment variables on your Linux host to embed identity data: 

```
export U=$(id -un)
export H=$(hostname -s)
export M=$(cat /etc/machine-id | head -c 16)
export T=$(date -u +%Y-%m-%dT%H:%M:%SZ)
export E=$(date +%s)
```
These variables represent your username, hostname, machine ID, build UTC timestamp, and epoch time.

![task2_1](https://github.com/user-attachments/assets/7f3d399d-4003-483a-be6c-89a2905bbd49)


### B. Common Header (unique.h)
You save a header file unique.h that includes these environment variables and a hash function to generate unique IDs. It also includes a function that prints a proof header with all this information plus the GCC version and program name whenever run.
```
#ifndef UNIQUE_H
#define UNIQUE_H
#include <stdio.h>
#include <stdint.h>
#include <time.h>
#ifndef PALAK_PARMAR
#define PALAK_PARMAR "PALAK_PARMAR"
#endif
#ifndef PDEU
#define PDEU "PDEU"
#endif
#ifndef MACHINE_ID
#define MACHINE_ID "unknown_machine"
#endif
#ifndef BUILD_UTC
#define BUILD_UTC "unknown_time"

#endif
#ifndef BUILD_EPOCH
#define BUILD_EPOCH 0
#endif
static uint64_t fnv1a64(const char *s) {
const uint64_t OFF = 1469598103934665603ULL, PRIME = 1099511628211ULL;
uint64_t h = OFF;
for (const unsigned char *p=(const unsigned char*)s; *p; ++p) {
h ^= *p; h *= PRIME;
}
return h;
}
static void uniq_print_header(const char *program_name) {
time_t now = time(NULL);
char buf[512];
int n = snprintf(buf, sizeof(buf), "%s|%s|%s|%s|%ld|%s|%s",
PALAK_PARMAR, PDEU, MACHINE_ID, BUILD_UTC,
(long)BUILD_EPOCH, __VERSION__, program_name);
(void)n;
uint64_t proof = fnv1a64(buf);
char rbuf[600];
snprintf(rbuf, sizeof(rbuf), "%s|run_epoch=%ld", buf, (long)now);
uint64_t runid = fnv1a64(rbuf);
printf("=== RISC-V Proof Header ===\n");
printf("User : %s\n", PALAK_PARMAR);
printf("Host : %s\n", PDEU);
printf("MachineID : %s\n", MACHINE_ID);
printf("BuildUTC : %s\n", BUILD_UTC);
printf("BuildEpoch : %ld\n", (long)BUILD_EPOCH);
printf("GCC : %s\n", __VERSION__);
printf("PointerBits: %d\n", (int)(8*(int)sizeof(void*)));
printf("Program : %s\n", program_name);
printf("ProofID : 0x%016llx\n", (unsigned long long)proof);
printf("RunID : 0x%016llx\n", (unsigned long long)runid);
printf("===========================\n");
}
#endif
```

### C. Programs to Implement
You implement 3-4 sample C programs including unique.h and calling the header printing function at the start. The example programs are:

#### 1.factorial.c 
calculates factorial of a number.
```
 #include "unique.h"
 static unsigned long long fact(unsigned n){ return (n<2)?1ULL:n*fact(n-1); }
 int main(void){
 uniq_print_header("factorial");
 unsigned n = 12;
 printf("n=%u, n!=%llu\n", n, fact(n));
 return 0;
 }
```

#### 2. max_array.c 
 finds the maximum value in an array.
```
 #include "unique.h"
 int main(void){
 uniq_print_header("max_array");
 int a[] = {42,-7,19,88,3,88,5,-100,37};
 int n = sizeof(a)/sizeof(a[0]), max=a[0];
 for(int i=1;i<n;i++) if(a[i]>max) max=a[i];
 printf("Array length=%d, Max=%d\n", n, max);
 return 0;
 }
```

 #### 3. bitops.c
 ```
 #include "unique.h"
 int main(void){
 uniq_print_header("bitops");
 unsigned x=0xA5A5A5A5u, y=0x0F0F1234u;
 printf("x&y=0x%08X\n", x&y);
 printf("x|y=0x%08X\n", x|y);
 printf("x^y=0x%08X\n", x^y);
 printf("x<<3=0x%08X\n", x<<3);
 printf("y>>2=0x%08X\n", y>>2);
 return 0;
 }
```

#### 4. bubble_sort.c 
```
 #include "unique.h"
 void bubble(int *a,int n){ for(int i=0;i<n-1;i++) for(int j=0;j<n-1-i;j++) if(a[j]>a[j
 +1]){int t=a[j];a[j]=a[j+1];a[j+1]=t;} }
 int main(void){
 uniq_print_header("bubble_sort");
 int a[]={9,4,1,7,3,8,2,6,5}, n=sizeof(a)/sizeof(a[0]);
 bubble(a,n);
 printf("Sorted:"); for(int i=0;i<n;i++) printf(" %d",a[i]); puts("");
 return 0;
 }
```

### D. Build, run, and capture evidence
Compile each program embedding the unique variables as macros:

#### 1.factorial.c 
compiling the code -:
```
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
  -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
  -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
  factorial.c -o factorial
  ```
Run it on Spike:
```
spike pk ./factorial
```
Ouyput :

![factorial run](https://github.com/user-attachments/assets/2c6bd0d2-83bc-4434-9470-8c4497495573)

#### 2. max_array.c 
compiling the code -:
```
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
  -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
  -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
 max_array.c -o max_array
  ```
Run it on Spike:
```
spike pk ./max_array
```
Output :

![run max_array](https://github.com/user-attachments/assets/e47e8cda-c258-4928-8c4d-056c988a46af)

#### 3. bitops.c 
compiling the code -:
```
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
  -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
  -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
 bitops.c -o bitops
  ```
Run it on Spike:
```
spike pk ./bitops
```
Output :

![run bitops](https://github.com/user-attachments/assets/4e745874-1683-44c9-94ed-4e5c2c9e20e3)

#### 4. bubble_sort.c
compiling the code -:
```
riscv64-unknown-elf-gcc -O0 -g -march=rv64imac -mabi=lp64 \
  -DUSERNAME="\"$U\"" -DHOSTNAME="\"$H\"" -DMACHINE_ID="\"$M\"" \
  -DBUILD_UTC="\"$T\"" -DBUILD_EPOCH=$E \
 bubble_sort.c -o bubble_sort
  ```
Run it on Spike:
```
spike pk ./bubble_sort
```
Output :

![bubble_sort run](https://github.com/user-attachments/assets/a87f2c84-b2a2-41c9-980e-d38c7eaca59e)

### E. Produce assembly and disassembly

 #### 1. factorial.c
 
 ##### Assembly (.s):
 Generate assembly listing:
 ```
 riscv64-unknown-elf-gcc-O0-S factorial.c-o factorial.s
```

Disassembly of main only:
Produce disassembly of the main function (to decode instructions):
```
riscv64-unknown-elf-objdump -d ./factorial | sed -n '/<main>:/,/^$/p' | tee factorial_main_objdump.txt
```
Output :

![factorial main](https://github.com/user-attachments/assets/a685bfac-9f8e-4f96-92da-50546403ec9a)

#### 2. max_array.c
 
 ##### Assembly (.s):
 Generate assembly listing:
 ```
 riscv64-unknown-elf-gcc-O0-S max_array.c-o max_array.s
```

##### Disassembly of main only:
Produce disassembly of the main function (to decode instructions):
```
riscv64-unknown-elf-objdump -d ./max_array | sed -n '/<main>:/,/^$/p' | tee max_array_main_objdump.txt
```
Output :

![array main](https://github.com/user-attachments/assets/c72c16dd-a26e-424d-97a3-9d028aaf72c3)

#### 3. bitops.c
 
 ##### Assembly (.s):
 Generate assembly listing:
 ```
 riscv64-unknown-elf-gcc-O0-S max_array.c-o bitops.s
```

##### Disassembly of main only:
Produce disassembly of the main function (to decode instructions):
```
riscv64-unknown-elf-objdump -d ./bitops | sed -n '/<main>:/,/^$/p' | tee bitops_main_objdump.txt
```
Output :

![bitops main](https://github.com/user-attachments/assets/c4ac9c6b-2c32-4c33-950b-bde0531b7cfb)

#### 4. bubble_sort.c
 
 ##### Assembly (.s):
 Generate assembly listing:
 ```
 riscv64-unknown-elf-gcc-O0-S max_array.c-o bitops.s
```

##### Disassembly of main only:
Produce disassembly of the main function (to decode instructions):
```
riscv64-unknown-elf-objdump -d ./bitops | sed -n '/<main>:/,/^$/p' | tee bitops_main_objdump.txt
```
Output :

![main bubble sort](https://github.com/user-attachments/assets/d6e422cb-620e-4956-a141-9b57455a4c9e)

### 📁 Repository structure

```
riscv-task2-<username>/
├──README.md
├── instruction_decoding.md
├── unique.h
├── factorial.c
├── factorial.s
├── factorial_main_objdump.txt
├── max_array.c
├── max_array.s
├── max_array_main_objdump.txt
├── bitops.c
├── bitops.s
├── bitops_main_objdump.txt
├── bubble_sort.c
├── bubble_sort.s
└── bubble_sort_main_objdump.txt
``







