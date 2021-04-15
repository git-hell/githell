.text
_fputc:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movl (%rsi),%eax
	addl $-1,%eax
	movl %eax,(%rsi)
	cmpl $0,%eax
	jl L5
L4:
	movq 24(%rsi),%rax
	movq %rax,%rcx
	addq $1,%rax
	movq %rax,24(%rsi)
	movb %dil,(%rcx)
	movzbl %dil,%eax
	jmp L3
L5:
	call ___flushbuf
L3:
	popq %rbp
	ret
L11:
.globl _fputc
.globl ___flushbuf
