.text
_fputs:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
L16:
	movq %rsi,%r12
	movq %rdi,%rbx
	xorl %r13d,%r13d
L4:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L6
L5:
	movl (%r12),%esi
	addl $-1,%esi
	movl %esi,(%r12)
	cmpl $0,%esi
	jl L11
L10:
	movzbl (%rbx),%esi
	addq $1,%rbx
	movq 24(%r12),%rdi
	movq %rdi,%rax
	addq $1,%rdi
	movq %rdi,24(%r12)
	movb %sil,(%rax)
	movzbl %sil,%eax
	jmp L12
L11:
	movzbl (%rbx),%edi
	addq $1,%rbx
	movq %r12,%rsi
	call ___flushbuf
L12:
	cmpl $-1,%eax
	jnz L8
L7:
	movl $-1,%eax
	jmp L3
L8:
	addl $1,%r13d
	jmp L4
L6:
	movl %r13d,%eax
L3:
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L18:
.globl _fputs
.globl ___flushbuf
