.text
_memcmp:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	cmpq $0,%rdx
	jz L6
L4:
	addq $1,%rdx
L7:
	addq $-1,%rdx
	jz L6
L8:
	movzbl (%rdi),%eax
	addq $1,%rdi
	movzbl (%rsi),%ecx
	addq $1,%rsi
	cmpl %ecx,%eax
	jz L7
L12:
	addq $-1,%rdi
	movzbl (%rdi),%eax
	addq $-1,%rsi
	movzbl (%rsi),%esi
	subl %esi,%eax
	jmp L3
L6:
	xorl %eax,%eax
L3:
	popq %rbp
	ret
L19:
.globl _memcmp
