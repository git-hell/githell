# memset.s - AMD64-specific memset()                       ncc standard library
#
#                    Copyright (c) 2021 Charles E. Youse (charles@gnuless.org).
#                       All rights reserved. See LICENSE file for more details.
#
# derived from public-domain version by J.T. Conklin <jtc@netbsd.org>
# and Open/NetBSD adaptation by Frank van der Linden <fvdl@wasabisytems.com>

.text

# void *memset(void *s, int c, size_t n);

.global _memset

_memset:        movq %rsi, %rax         # c
	            andq $0xff, %rax
	            movq %rdx, %rcx         # n
                movq %rdi, %r11         # s

	            # if the string is too short, it's really not worth
                # the overhead of aligning to word boundaries, etc.
                # so we jump to a plain unaligned set.

	            cmpq $0x0f, %rcx
	            jle	L1

	            movb %al,%ah			# copy char to all bytes in word
	            movl %eax,%edx
	            sall $16,%eax
	            orl	%edx,%eax
	            movl %eax,%edx
	            salq $32,%rax
	            orq	%rdx,%rax

	            movq %rdi, %rdx         # compute misalignment
	            negq %rdx
	            andq $7, %rdx
	            movq %rcx, %r8
	            subq %rdx, %r8

	            movq %rdx, %rcx         # set by bytes to alignment boundary
	            rep
	            stosb

	            movq %r8, %rcx
	            shrq $3, %rcx			# set by words
	            rep
	            stosq

	            movq %r8, %rcx		    # set remainder by bytes
	            andq $7, %rcx
L1:	            rep
	            stosb

	            movq %r11, %rax
                ret

# vi: set ts=4 expandtab:
