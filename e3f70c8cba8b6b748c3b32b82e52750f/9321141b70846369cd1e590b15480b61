.text
_gets:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L27:
	movq %rdi,%rbx
	movq %rbx,%r12
L4:
	movl ___stdin(%rip),%esi
	addl $-1,%esi
	movl %esi,___stdin(%rip)
	cmpl $0,%esi
	jl L12
L11:
	movq ___stdin+24(%rip),%rsi
	movq %rsi,%rdi
	addq $1,%rsi
	movq %rsi,___stdin+24(%rip)
	movzbl (%rdi),%eax
	jmp L13
L12:
	movq $___stdin,%rdi
	call ___fillbuf
L13:
	cmpl $-1,%eax
	jz L6
L7:
	cmpl $10,%eax
	jz L6
L5:
	movb %al,(%r12)
	addq $1,%r12
	jmp L4
L6:
	cmpl $-1,%eax
	jnz L16
L14:
	movl ___stdin+8(%rip),%esi
	testl $16,%esi
	jz L18
L17:
	cmpq %r12,%rbx
	jnz L16
L20:
	xorl %eax,%eax
	jmp L3
L18:
	xorl %eax,%eax
	jmp L3
L16:
	movb $0,(%r12)
	movq %rbx,%rax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L29:
.globl _gets
.globl ___fillbuf
.globl ___stdin
