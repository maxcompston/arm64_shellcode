# arm64_shellcode

Source code for mapping shell code into memory and executing the mapped shell code.
The arm64 code was tested on a Raspberry Pi running Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-1062-raspi2 aarch64).

## License

Please see the [LICENSE](LICENSE) file for the exact details.

## sc_mapped 

## About

The C program sc_mapped loads an array of arm-64 shell code into and memory mapped location.  After relocating the shell code this code is invoked as a C function.  The shell code takes over invoking a system call to execve a /bin/sh.

Now the shell code assembler source code has been modified to directly perform the system call by substituting the execve with the linux system call number.  Additionally, the null bytes have been removed from the code.  This will allow the shell code to be used to exploit memory corruption vulnerabilities.

This program can be ran as a shell or root user.

target $ sudo -s
target #
shell # ./sc_mapped 
Shellcode Length: 40 Bytes
shell # whoami
root
shell id
uid=0(root) gid=0(root) groups=0(root)
shell # exit
target #

## Stack Management Issues

For AArch64, sp must be 16-byte aligned whenever it is used to access memory. This is enforced by AArch64 hardware.

// Broken AArch64 implementation of `push {x1}; push {x0};`.
str   x1, [sp, #-8]!  // This works, but leaves `sp` with only 8-byte alignment
str   x0, [sp, #-8]!  // ... so the second `str` will fail.

C compilers will typically reserve stack space at the start of the function, then leave sp alone until the end, so the restriction is not as awkward as it first seems. However, you must be aware of it when handling assembly code.

It sounds wasteful – and in most cases it is – but the simplest way to handle the stack pointer can be to push each value to a 16-byte slot. This doubles the stack usage (for pointer-sized values), and it effectively reduces the available memory bandwidth. It is also awkward to implement multiple-register operations using this scheme, since each register requires a separate instruction.

## Sources: sc_mapped

Credit to Ken Kitahara for the use of his source code.

https://www.exploit-db.com/exploits/47048

Max Compston, Embedded Software Solutions.

## Building this Release

The source code for this program is built using CMake.  

The source code for the arm64 target are cross compiled using gnu arm64 compiler, below.  First in install the cross-compiler on the host.

$ sudo apt-get install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

Next, set up the host to cross-compile arm64 creating a toolchain-aarch64-linux-gnu.cmake file in your home directory.  Add the following:

#- this one is important
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_PROCESSOR arm64)

#- specify the cross compiler
SET(CMAKE_C_COMPILER   /usr/bin/aarch64-linux-gnu-gcc)
SET(CMAKE_CXX_COMPILER /usr/bin/aarch64-linux-gnu-g++)

#- where is the target environment
SET(CMAKE_FIND_ROOT_PATH  /user/aarch64-linux-gnu)

#- search for programs in the build host directories
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

#- for libraries and headers in the target directories
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

To build the cross-compiled arm64 program navigate to the build directory and run cmake as follows:

$ cmake -DCMAKE_TOOLCHAIN_FILE=~/toolchain-aarch64-linux-gnu.cmake .. && make
