.text
_ftell:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
L2:
	movl 8(%rdi),%eax
	testl $128,%eax
	jz L5
L4:
	movl (%rdi),%ebx
	negl %ebx
	jmp L6
L5:
	testl $256,%eax
	jz L8
L14:
	movq 16(%rdi),%rsi
	cmpq $0,%rsi
	jz L8
L10:
	testl $4,%eax
	jnz L8
L7:
	movq 24(%rdi),%rbx
	subq %rsi,%rbx
	jmp L6
L8:
	xorl %ebx,%ebx
L6:
	movl 4(%rdi),%edi
	xorl %esi,%esi
	movl $1,%edx
	call _lseek
	cmpq $-1,%rax
	jnz L20
L18:
	movq $-1,%rax
	jmp L3
L20:
	movslq %ebx,%rsi
	addq %rsi,%rax
L3:
	popq %rbx
	popq %rbp
	ret
L26:
.globl _ftell
.globl _lseek
