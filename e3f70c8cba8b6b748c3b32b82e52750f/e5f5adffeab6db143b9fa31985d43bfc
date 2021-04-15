.text
_bsearch:
L1:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L16:
	movq %r8,%r15
	movq %rcx,%r14
	movq %rdi,-8(%rbp)	 # spill
	movq %rdx,%rbx
	movq %rsi,%r12
L4:
	cmpq $0,%rbx
	jz L6
L5:
	movq %rbx,%rsi
	shrq $1,%rsi
	movq %r14,%rdi
	imulq %rsi,%rdi
	leaq (%r12,%rdi),%r13
	movq -8(%rbp),%rdi	 # spill
	movq %r13,%rsi
	call *%r15
	cmpl $0,%eax
	jnz L9
L7:
	movq %r13,%rax
	jmp L3
L9:
	cmpl $0,%eax
	jl L12
L11:
	leaq (%r13,%r14),%r12
	addq $-1,%rbx
	shrq $1,%rbx
	jmp L4
L12:
	shrq $1,%rbx
	jmp L4
L6:
	xorl %eax,%eax
L3:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L18:
.globl _bsearch
