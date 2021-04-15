.text
_fgetpos:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L10:
	movq %rsi,%rbx
	call _ftell
	movq %rax,(%rbx)
	cmpq $-1,%rax
	jnz L5
L4:
	movl $-1,%eax
	jmp L3
L5:
	xorl %eax,%eax
L3:
	popq %rbx
	popq %rbp
	ret
L12:
.globl _fgetpos
.globl _ftell
