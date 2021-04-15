.text
.align 8
_one:
	.quad 0x3ff0000000000000
.align 8
L63:
	.quad 0x0
_modf:
L2:
	pushq %rbp
	movq %rsp,%rbp
	subq $80,%rsp
L5:
	movsd %xmm0,-8(%rbp)
	movl -4(%rbp),%eax
	movl -8(%rbp),%esi
	movl %eax,%edx
	sarl $20,%edx
	andl $2047,%edx
	leal -1023(%rdx),%ecx
	cmpl $20,%ecx
	jge L9
L8:
	cmpl $0,%ecx
	jge L12
L14:
	andl $2147483648,%eax
	movl %eax,-12(%rbp)
	movl $0,-16(%rbp)
	movsd -16(%rbp),%xmm1
	movsd %xmm1,(%rdi)
	jmp L4
L12:
	movl $1048575,%edx
	shrl %cl,%edx
	movl %eax,%ecx
	andl %edx,%ecx
	orl %esi,%ecx
	cmpl $0,%ecx
	jnz L28
L18:
	movsd %xmm0,(%rdi)
	movsd %xmm0,-24(%rbp)
	movl -20(%rbp),%esi
	andl $2147483648,%esi
	movl %esi,-28(%rbp)
	movl $0,-32(%rbp)
	movsd -32(%rbp),%xmm0
	jmp L4
L28:
	notl %edx
	andl %edx,%eax
	movl %eax,-36(%rbp)
	movl $0,-40(%rbp)
	movsd -40(%rbp),%xmm1
	movsd %xmm1,(%rdi)
	subsd %xmm1,%xmm0
	jmp L4
L9:
	cmpl $51,%ecx
	jle L33
L32:
	cmpl $1024,%ecx
	jnz L37
L35:
	movsd %xmm0,(%rdi)
	movsd L63(%rip),%xmm1
	divsd %xmm0,%xmm1
	movsd %xmm1,%xmm0
	jmp L4
L37:
	movsd _one(%rip),%xmm2
	movsd %xmm0,%xmm1
	mulsd %xmm2,%xmm1
	movsd %xmm1,(%rdi)
	movsd %xmm0,-48(%rbp)
	movl -44(%rbp),%esi
	andl $2147483648,%esi
	movl %esi,-52(%rbp)
	movl $0,-56(%rbp)
	movsd -56(%rbp),%xmm0
	jmp L4
L33:
	leal -1043(%rdx),%ecx
	movl $4294967295,%edx
	shrl %cl,%edx
	testl %edx,%esi
	jnz L56
L46:
	movsd %xmm0,(%rdi)
	movsd %xmm0,-64(%rbp)
	movl -60(%rbp),%esi
	andl $2147483648,%esi
	movl %esi,-68(%rbp)
	movl $0,-72(%rbp)
	movsd -72(%rbp),%xmm0
	jmp L4
L56:
	movl %eax,-76(%rbp)
	notl %edx
	andl %edx,%esi
	movl %esi,-80(%rbp)
	movsd -80(%rbp),%xmm1
	movsd %xmm1,(%rdi)
	subsd %xmm1,%xmm0
L4:
	movq %rbp,%rsp
	popq %rbp
	ret
L64:
.globl _modf
