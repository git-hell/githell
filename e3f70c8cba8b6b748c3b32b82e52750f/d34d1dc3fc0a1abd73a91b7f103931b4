.text
_puts:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L23:
	movq %rdi,%rbx
	xorl %r12d,%r12d
L4:
	movzbl (%rbx),%esi
	cmpl $0,%esi
	jz L6
L5:
	movl ___stdout(%rip),%esi
	addl $-1,%esi
	movl %esi,___stdout(%rip)
	cmpl $0,%esi
	jl L11
L10:
	movzbl (%rbx),%esi
	movq ___stdout+24(%rip),%rdi
	addq $1,%rbx
	movq %rdi,%rax
	addq $1,%rdi
	movq %rdi,___stdout+24(%rip)
	movb %sil,(%rax)
	movzbl %sil,%eax
	jmp L12
L11:
	movzbl (%rbx),%edi
	addq $1,%rbx
	movq $___stdout,%rsi
	call ___flushbuf
L12:
	cmpl $-1,%eax
	jnz L8
L7:
	movl $-1,%eax
	jmp L3
L8:
	addl $1,%r12d
	jmp L4
L6:
	movl ___stdout(%rip),%esi
	addl $-1,%esi
	movl %esi,___stdout(%rip)
	cmpl $0,%esi
	jl L18
L17:
	movq ___stdout+24(%rip),%rsi
	movq %rsi,%rdi
	addq $1,%rsi
	movq %rsi,___stdout+24(%rip)
	movb $10,(%rdi)
	jmp L16
L18:
	movl $10,%edi
	movq $___stdout,%rsi
	call ___flushbuf
	cmpl $-1,%eax
	jnz L16
L14:
	movl $-1,%eax
	jmp L3
L16:
	leal 1(%r12),%eax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L25:
.globl ___stdout
.globl _puts
.globl ___flushbuf
