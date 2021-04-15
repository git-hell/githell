.text
_fseek:
L1:
	pushq %rbp
	movq %rsp,%rbp
	pushq %rbx
	pushq %r12
	pushq %r13
	pushq %r14
L33:
	movq %rsi,%r13
	movq %rdi,%rbx
	movl %edx,%r12d
	xorl %r14d,%r14d
	movl 8(%rbx),%esi
	andl $-49,%esi
	movl %esi,8(%rbx)
	testl $128,%esi
	jz L5
L4:
	cmpl $1,%r12d
	jnz L9
L14:
	cmpq $0,16(%rbx)
	jz L9
L10:
	testl $4,%esi
	jnz L9
L7:
	movl (%rbx),%r14d
L9:
	movl $0,(%rbx)
	jmp L6
L5:
	testl $256,%esi
	jz L6
L18:
	movq %rbx,%rdi
	call _fflush
L6:
	movslq %r14d,%rsi
	subq %rsi,%r13
	movl 4(%rbx),%edi
	movq %r13,%rsi
	movl %r12d,%edx
	call _lseek
	movl 8(%rbx),%esi
	testl $1,%esi
	jz L23
L24:
	testl $2,%esi
	jz L23
L21:
	andl $-385,%esi
	movl %esi,8(%rbx)
L23:
	movq 16(%rbx),%rsi
	movq %rsi,24(%rbx)
	cmpq $-1,%rax
	jnz L29
L28:
	movl $-1,%eax
	jmp L3
L29:
	xorl %eax,%eax
L3:
	popq %r14
	popq %r13
	popq %r12
	popq %rbx
	popq %rbp
	ret
L35:
.globl _fflush
.globl _lseek
.globl _fseek
