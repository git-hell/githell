.text
_fwrite:
L1:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
	pushq %r15
L22:
	movq %rsi,-8(%rbp)	 # spill
	movq %rcx,%r12
	movq %rdi,%rbx
	movq %rdx,%r15
	xorl %r14d,%r14d
	cmpq $0,-8(%rbp)	 # spill
	jz L6
L7:
	cmpq %r15,%r14
	jae L6
L8:
	movq -8(%rbp),%r13	 # spill
L10:
	movl (%r12),%esi
	addl $-1,%esi
	movl %esi,(%r12)
	cmpl $0,%esi
	jl L17
L16:
	movzbl (%rbx),%esi
	movq 24(%r12),%rdi
	movq %rdi,%rax
	addq $1,%rdi
	movq %rdi,24(%r12)
	movb %sil,(%rax)
	movzbl %sil,%eax
	jmp L18
L17:
	movzbl (%rbx),%edi
	movq %r12,%rsi
	call ___flushbuf
L18:
	cmpl $-1,%eax
	jnz L15
L13:
	movq %r14,%rax
	jmp L3
L15:
	addq $1,%rbx
	addq $-1,%r13
	jnz L10
L11:
	addq $1,%r14
	jmp L7
L6:
	movq %r14,%rax
L3:
	popq %r15
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	movq %rbp,%rsp
	popq %rbp
	ret
L24:
.globl _fwrite
.globl ___flushbuf
