.text
_getenv:
L1:
	pushq %rbp
	movq %rsp,%rbp
L2:
	movq _environ(%rip),%rsi
	movq %rsi,%rcx
	cmpq $0,%rsi
	jz L4
L7:
	cmpq $0,%rdi
	jnz L12
L4:
	xorl %eax,%eax
	jmp L3
L12:
	movq (%rcx),%rax
	addq $8,%rcx
	movq %rax,%rsi
	cmpq $0,%rax
	jz L14
L13:
	movq %rdi,%r8
L15:
	movzbl (%r8),%eax
	cmpl $0,%eax
	jz L17
L18:
	movzbl (%rsi),%edx
	addq $1,%rsi
	cmpl %edx,%eax
	jnz L17
L16:
	addq $1,%r8
	jmp L15
L17:
	movzbl (%r8),%eax
	cmpl $0,%eax
	jnz L12
L25:
	movzbl (%rsi),%eax
	cmpl $61,%eax
	jnz L12
L24:
	leaq 1(%rsi),%rax
	jmp L3
L14:
	xorl %eax,%eax
L3:
	popq %rbp
	ret
L35:
.globl _getenv
.globl _environ
