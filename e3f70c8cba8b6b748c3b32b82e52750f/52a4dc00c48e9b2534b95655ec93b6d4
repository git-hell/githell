.local L4
.comm L4, 20, 1
.text
___fillbuf:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
L60:
	movq %rdi,%rbx
	movl $0,(%rbx)
	movl 4(%rbx),%esi
	cmpl $0,%esi
	jge L7
L5:
	movl $-1,%eax
	jmp L3
L7:
	movl 8(%rbx),%esi
	testl $48,%esi
	jz L11
L9:
	movl $-1,%eax
	jmp L3
L11:
	testl $1,%esi
	jnz L15
L13:
	orl $32,%esi
	movl %esi,8(%rbx)
	movl $-1,%eax
	jmp L3
L15:
	testl $256,%esi
	jz L19
L17:
	orl $32,%esi
	movl %esi,8(%rbx)
	movl $-1,%eax
	jmp L3
L19:
	testl $128,%esi
	jnz L23
L21:
	orl $128,%esi
	movl %esi,8(%rbx)
L23:
	movl 8(%rbx),%esi
	testl $4,%esi
	jnz L26
L27:
	cmpq $0,16(%rbx)
	jnz L26
L24:
	movl $1024,%edi
	call _malloc
	movq %rax,16(%rbx)
	cmpq $0,%rax
	jnz L32
L31:
	orl $4,8(%rbx)
	jmp L26
L32:
	orl $8,8(%rbx)
	movl $1024,12(%rbx)
L26:
	xorl %r12d,%r12d
L35:
	movslq %r12d,%rsi
	movq ___iotab(,%rsi,8),%rdi
	cmpq $0,%rdi
	jz L36
L41:
	movl 8(%rdi),%esi
	testl $64,%esi
	jz L36
L38:
	testl $256,%esi
	jz L36
L45:
	call _fflush
L36:
	addl $1,%r12d
	cmpl $20,%r12d
	jl L35
L37:
	cmpq $0,16(%rbx)
	jnz L50
L48:
	movl 4(%rbx),%esi
	movslq %esi,%rsi
	addq $L4,%rsi
	movq %rsi,16(%rbx)
	movl $1,12(%rbx)
L50:
	movq 16(%rbx),%rsi
	movq %rsi,24(%rbx)
	movl 12(%rbx),%esi
	movslq %esi,%rdx
	movq 16(%rbx),%rsi
	movl 4(%rbx),%edi
	call _read
	movl %eax,(%rbx)
	cmpl $0,%eax
	jg L53
L51:
	cmpl $0,%eax
	jnz L55
L54:
	orl $16,8(%rbx)
	jmp L56
L55:
	orl $32,8(%rbx)
L56:
	movl $-1,%eax
	jmp L3
L53:
	leal -1(%rax),%esi
	movl %esi,(%rbx)
	movq 24(%rbx),%rsi
	movq %rsi,%rdi
	addq $1,%rsi
	movq %rsi,24(%rbx)
	movzbl (%rdi),%eax
L3:
	popq %r12
	popq %rbx
	popq %rbp
	ret
L62:
.globl ___iotab
.globl _malloc
.globl _read
.globl ___fillbuf
.globl _fflush
