.text
_sbrk:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L13:
	movq %rdi,%rbx
	xorl %edi,%edi
	call ___brk
	movq %rax,%r12
	cmpq $0,%rbx
	jz L6
L4:
	leaq (%r12,%rbx),%rdi
	call ___brk
	cmpq %rax,%r12
	jnz L6
L7:
	movl $12,_errno(%rip)
	movq $-1,%rax
	jmp L3
L6:
	movq %r12,%rax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L15:
_brk:
L16:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L25:
	movq %rdi,%rbx
	call ___brk
	cmpq %rbx,%rax
	jae L21
L19:
	movl $12,_errno(%rip)
	movl $-1,%eax
	jmp L18
L21:
	xorl %eax,%eax
L18:
	popq %rbx
	popq %rbp
	ret
L27:
.globl _sbrk
.globl _brk
.globl ___brk
.globl _errno
