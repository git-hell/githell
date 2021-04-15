# crt0.s - AMD64 c runtime entry point                     ncc standard library
#
#                    Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).
#                       All rights reserved. See LICENSE file for more details.

.text

.global cstart
.global _exit

cstart:         popq %rdi                   # argc
                movq %rsp, %rsi             # argv
                leaq 8(%rsi,%rdi,8), %rdx   # envp

                movq %rdx, _environ(%rip)
                cld
                call _main

                movl %eax, %edi             # pass along main's return value
                call _exit                  # to exit()

.comm _environ, 8, 8

# vi: set ts=4 expandtab:
