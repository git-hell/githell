.text
_frexp:
L1:
	pushq %rbp
	movq %rsp,%rbp
	subq $8,%rsp
L2:
	movsd %xmm0,-8(%rbp)
	movl -4(%rbp),%esi
	shll $1,%esi
	shrl $21,%esi
	cmpl $0,%esi
	jz L8
L19:
	cmpl $2047,%esi
	jz L6
L5:
	movl -4(%rbp),%esi
	shll $1,%esi
	shrl $21,%esi
	addl $-1022,%esi
	movl %esi,(%rdi)
	movl -4(%rbp),%esi
	andl $2148532223,%esi
	orl $1071644672,%esi
	movl %esi,-4(%rbp)
	jmp L6
L8:
	movl -8(%rbp),%eax
	movl -4(%rbp),%esi
	shll $12,%esi
	shrl $12,%esi
	orl %esi,%eax
	cmpl $0,%eax
	jnz L10
L9:
	movl $0,(%rdi)
	jmp L6
L10:
	movsd ___frexp_adj(%rip),%xmm1
	mulsd %xmm1,%xmm0
	movsd %xmm0,-8(%rbp)
	movl -4(%rbp),%esi
	shll $1,%esi
	shrl $21,%esi
	addl $-1536,%esi
	movl %esi,(%rdi)
	movl -4(%rbp),%esi
	andl $2148532223,%esi
	orl $1071644672,%esi
	movl %esi,-4(%rbp)
L6:
	movsd -8(%rbp),%xmm0
L3:
	movq %rbp,%rsp
	popq %rbp
	ret
L23:
.globl ___frexp_adj
.globl _frexp
