.text
_memcpy:
L1:
	pushq %rbp
	movq %rsp,%rbp
L12:
	movq %rdi,%rax
	cmpq $0,%rdx
	jz L3
L4:
	addq $1,%rdx
L7:
	addq $-1,%rdx
	jz L3
L8:
	movzbl (%rsi),%ecx
	addq $1,%rsi
	movb %cl,(%rdi)
	addq $1,%rdi
	jmp L7
L3:
	popq %rbp
	ret
L14:
.globl _memcpy
