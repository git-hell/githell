.text
_fgets:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L31:
	movl %esi,%r12d
	movq %rdx,%r13
	movq %rdi,%r14
	movq %r14,%rbx
L4:
	addl $-1,%r12d
	cmpl $0,%r12d
	jle L6
L7:
	movl (%r13),%esi
	addl $-1,%esi
	movl %esi,(%r13)
	cmpl $0,%esi
	jl L12
L11:
	movq 24(%r13),%rsi
	movq %rsi,%rdi
	addq $1,%rsi
	movq %rsi,24(%r13)
	movzbl (%rdi),%eax
	jmp L13
L12:
	movq %r13,%rdi
	call ___fillbuf
L13:
	movl %eax,%ecx
	cmpl $-1,%eax
	jz L6
L5:
	movb %al,(%rbx)
	addq $1,%rbx
	cmpl $10,%eax
	jnz L4
L6:
	cmpl $-1,%ecx
	jnz L20
L18:
	movl 8(%r13),%esi
	testl $16,%esi
	jz L22
L21:
	cmpq %rbx,%r14
	jnz L20
L24:
	xorl %eax,%eax
	jmp L3
L22:
	xorl %eax,%eax
	jmp L3
L20:
	movb $0,(%rbx)
	movq %r14,%rax
L3:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L33:
.globl _fgets
.globl ___fillbuf
